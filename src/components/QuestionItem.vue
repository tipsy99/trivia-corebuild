<template>
  <div class="my-3">
    <div class="card text-dark bg-light" :class="question.submitted ? `border-${question.submitted == question.correct_answer ? 'success' : 'danger'}` : '' ">
      <div class="card-body">
        <h5 class="card-title text-start" v-html="question.question"></h5>
        <div class="row">
          <h6 class="card-subtitle text-start col-6"><span class="badge bg-secondary me-2">Category</span>{{question.category}}</h6>
          <h6 class="card-subtitle text-end col-6"><span class="badge bg-secondary me-2">Difficulty</span>{{question.difficulty}}</h6>
        </div>
        <div class="d-flex justify-content-evenly flex-wrap mt-3" v-if="!question.submitted">
          <label v-for="answer, index in answers" :key="index" class="me-3">
            <input type="radio" :value="answer" v-model="selectedAnswer">
            <span v-html="answer"></span>
          </label>
        </div>
        <div v-else>
          <div class="row">
            <p class="col-6">
              Submitted answer: {{question.submitted}}
            </p>
            <p class="col-6">
              Correct answer: {{question.correct_answer}}
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */

export default {
  props: {
    question: {type: Object, required: true}
  },
  data() {
    return {
      selectedAnswer: null
    }
  },
  computed: {
    answers() {
      const vm = this
      if(vm.question.type === 'boolean') {
        return ['True', 'False']
      } else {
        return [...vm.question.incorrect_answers, vm.question.correct_answer]
      }
    }
  },
  watch: {
    selectedAnswer(newVal) {
      this.$emit('input', newVal)
    }
  }
}
</script>