<template>
  <div>
    <h1>
      TRIVIA GAME
    </h1>
    <button class="btn btn-primary" @click="startGame" v-if="!gameInProgress">START GAME</button>
    <GameScreen v-else/>
  </div>    
</template>

<script>
import GameScreen from './GameScreen.vue'
export default {
  components: { 
    GameScreen
  },
  data() {
    return {
      gameInProgress: false
    }
  },
  methods: {
    startGame() {
      this.gameInProgress = true
    }
  }
}
</script>