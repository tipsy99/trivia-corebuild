<template>
  <div class="container">
    <div class="row" v-if="!gameFinished">
      <div class="col-6 d-flex justify-content-end">
        <h5>Dummy Points <span class="badge bg-danger">{{dummyPoints}}</span></h5>
      </div>
      <div class="col-6 d-flex justify-content-start">
        <h5><span class="badge bg-success">{{brilliantPoints}}</span> Brilliant Points</h5>
      </div>
    </div>
    <div v-if="!gameFinished && gameLoaded && currentQuestion">
      <div class="progress">
        <div class="progress-bar progress-bar-striped bg-danger progress-bar-animated" role="progressbar" :style="`width: ${progressLength}%`" aria-valuenow="30" aria-valuemin="0" aria-valuemax="30">{{countdown}}</div>
      </div>
      <QuestionItem :question="currentQuestion" v-model="answer"/>
      <button class="btn btn-success" @click="handleSubmit" :disabled="!answer">Submit answer</button>
    </div>
    <div v-if="gameFinished">
      <div class="row">
        <div class="col-12">
          <h4>Questions answered correctly: {{brilliantPoints}}</h4>
        </div>
        <div class="col-12">
          <h4>Past questions</h4>
        </div>
        <div class="col-lg-6" v-for="question, index in pastQuestions" :key="index">
          <QuestionItem :question="question"/>
        </div>
      </div>
      <button class="btn btn-info" @click="startGame">Restart game</button>
    </div>
  </div>
</template>

<script>
import QuestionItem from './QuestionItem.vue'

export default {
  components: {
    QuestionItem
  },
  data() {
    return {
      token: '',
      questions: [],
      currentQuestion: {},
      gameLoaded: false,
      countdown: null,
      answer: null,
      countdownTimeout: null,
      brilliantPoints: 0,
      dummyPoints: 0,
      pastQuestions: [],
      gameFinished: false,
    }
  },
  methods: {
    async fetchQuestions() {
      this.gameLoaded = false
      let response = await fetch("https://opentdb.com/api.php?amount=10&category=18&difficulty=medium&token" + this.token)
      response = await response.json()
      this.questions = response.results
    },
    async fetchToken() {
      let response = await fetch("https://opentdb.com/api_token.php?command=request");
      response = await response.json()
      this.token = response.token
    }, 
    async getQuestion() {
      const vm = this
      vm.gameLoaded = false
      if(vm.questions.length) {
        vm.currentQuestion = vm.questions[0]
        vm.questions.splice(0, 1)
      } else {
        await vm.fetchQuestions()
      }
      vm.gameLoaded = true
      vm.countdown = 30
    },
    async handleSubmit() {
      const vm = this
      clearTimeout(vm.countdownTimeout)
      if(vm.answer === vm.currentQuestion.correct_answer) {
        vm.brilliantPoints++
      } else {
        vm.dummyPoints++
      }
      vm.pastQuestions.push({...vm.currentQuestion, submitted: vm.answer})
      vm.answer = null
      if(vm.brilliantPoints - vm.dummyPoints > 0) {
        await vm.getQuestion()
      } else {
        vm.gameFinished = true
      }
    },
    async startGame() {
      this.gameLoaded = false
      this.currentQuestion = {}
      this.gameFinished = false
      this.brilliantPoints = 0
      this.dummyPoints = 0
      this.pastQuestions = []
      await this.fetchToken()
      await this.fetchQuestions()
      await this.getQuestion()
    },
  },
  mounted() {
    this.startGame()
  },
  computed: {
    progressLength() {
      return Math.round(this.countdown / 30 * 100)
    }
  },
  watch: {
    countdown(value) {
      if(value > 0 && this.gameLoaded) {
        this.countdownTimeout = setTimeout(() => {
          this.countdown--
        }, 1000)
      } else if(value > 0) {
        this.gameFinished = true
        clearTimeout(this.countdownTimeout)
      }
    }, 
    gameLoaded(value) {
      if(value) {
        this.countdown = 30
      }
    }
  }
}
</script>