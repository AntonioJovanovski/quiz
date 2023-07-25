<template>
  <button v-if="startGameCondition" @click="startGame">
    <h1>START</h1>
  </button>
  <button v-if="round === 16" @click="startGame">
    <h1>YOU WON! Start again?</h1>
  </button>
  
  <div v-if="questionsFinished">You answered all answers correctly</div>
  <div v-if="loading">
    <span class="loader"></span>

  </div>
  <div v-if="renderMainScreen">
    <h4 v-html="replaceQuotes(question)"></h4>
    <h5>{{ roundLogic }}</h5>
    <ul>
      <li 
      class="help"
      v-if="help" 
      @click="showAnswer">{{ replaceQuotes(help) }}</li>

      <li 
      v-for="(answer, index) in randomAnswers" 
      :key="answer" 
      @click="chooseAnswer(answer, index)">{{ replaceQuotes(answer) }}</li>
      <li 
      class="correct" 
      v-if="answerSubmit">{{ finalAnswer }}</li>

      <button 
      class="help" 
      @click="removeOne" 
      :disabled="disableHelp">{{ helpButtonComputed }}</button>

    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      answers: [],
      correctAnswer: '',
      question: '',
      randomAnswers: [],
      round: 0,
      answerClick: true,
      answerSubmit: false,
      loading: false,
      won: null,
      help: false,
      questionsFinished: false,
      startGameCondition: true,
      renderMainScreen: false,
      disableHelp: false
    }
  },
  methods: {
    async callApi() {
      try {
        this.loading = true
        this.renderMainScreen = false
        this.help = false
        const response = await fetch(`https://opentdb.com/api.php?amount=1&difficulty=${this.apiLogic}&type=multiple`)
        const data = await response.json()
        this.answers.push(data.results[0].incorrect_answers)
        this.correctAnswer = data.results[0].correct_answer
        this.answers[0].push(data.results[0].correct_answer)
        this.question = data.results[0].question
        this.randomizeAnswers()
        this.round++
        this.answerClick = true
        this.loading = false
        this.answerSubmit = false
        this.renderMainScreen = true
      } catch (error) {
        alert(error)
      }
    },
    startGame() {
      this.loading = true
      this.startGameCondition = false
      this.round = 0
      this.callApi()
    },
    replaceQuotes(text) {
      return text
        .replace(/&quot;/g, '"')
        .replace(/&#039;/g, "'")
        .replace(/&amp;/g, "&");
    },
    randomizeAnswers() {
      const answers = this.answers[0]
      for (let i = answers.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        const temp = answers[i]
        answers[i] = answers[j]
        answers[j] = temp
      }
      this.randomAnswers = answers
    },
    chooseAnswer(answer, index) {
      if (answer === this.correctAnswer && this.answerClick) {
        this.won = true
        this.answers = []
        this.answerClick = false
        if (this.round !== 16) {
          setTimeout(() => {
            this.callApi()
          }, 1000);
        }
      } else if (answer !== this.correctAnswer && this.answerClick) {
        this.won = false
        this.answers = []
        this.answerClick = false
        setTimeout(() => {
          this.renderMainScreen = false
          this.round = 0
          this.disableHelp = false
          this.callApi()
        }, 1000);
      }
      this.answerSubmit = true
    },
    showAnswer() {
      this.help = !this.help
    },
    removeOne() {
      const incorrectAnswerIndex = this.randomAnswers.findIndex((item) => item !== this.correctAnswer);
      if (incorrectAnswerIndex !== -1) {
        this.randomAnswers.splice(incorrectAnswerIndex, 1);
      }
      this.disableHelp = true
    }
  },
  computed: {
    roundLogic() {
      if (this.round <= 5) {
        return "Round " + this.round + " of 5. Easy"
      } else if (this.round <= 10) {
        return "Round " + this.round + " of 10. Medium"
      } else {
        return "Round " + this.round + " of 15. Hard"
      }
    },
    apiLogic() {
      if (this.round <= 5) {
        return 'easy'
      } else if (this.round <= 10) {
        return 'medium'
      } else {
        return 'hard'
      }
    },
    finalAnswer() {
      if (this.won) {
        return "Correct answer"
      } else {
        return "Wrong answer, starting from beggingin"
      }
    },
    help() {
      if (this.help) {
        return this.correctAnswer
      } else {
        return 'Show Answer'
      }
    },
    helpButtonComputed() {
      if (!this.disableHelp) {
        return 'Help (remove one incorrect item)'
      } else {
        return 'Help available once in each of the 3 sections'
      }
    }
  },
  watch: {
    round(num) {
      if (num === 16) {
        this.renderMainScreen = false
      }
      if (num === 6) {
        this.disableHelp = false
      } else if (num === 11) {
        this.disableHelp = false
      }
    }
  },

}
</script>

<style>
* {
  transition: all 1s;
  transition: width 1s;
}

body {
  overflow: hidden;
  background-color: black;
  font-family: Arial, Helvetica, sans-serif;
}

.green {
  background-color: green;
}

div {
  color: white;
  display: flex;
  width: 100vw;
  height: 100vh;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

ul {
  padding: 0;
  text-align: center;
}

li {
  list-style: none;
  border: 1px solid white;
  padding: 10px 20px;
  margin: 2px;
  text-align: center;
  max-width: 300px;

}

li:active {
  scale: .95;
}

li:hover {
  cursor: pointer;
}

.loader {
  width: 48px;
  height: 48px;
  border: 5px solid #FFF;
  border-bottom-color: transparent;
  border-radius: 50%;
  display: inline-block;
  box-sizing: border-box;
  animation: rotation 1s linear infinite;
}

@keyframes rotation {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

button {
  cursor: pointer;
}

.help {
  margin: 10px;
  border: 1px solid red;
  padding: 10px 20px;
  background: none;
  color: white;
  width: auto;

}

.correct {
  list-style: none;
  border: 1px solid blue;
  padding: 10px 20px;
  margin: 2px;

  text-align: center;
  max-width: 300px;

}
</style>