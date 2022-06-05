<template>
  <div v-if="this.question">
  <ScoreBoard :winCount="winCount" :loseCount="loseCount" />
    <h1 v-html="question"></h1>
    <div v-for="(answer, index) in this.answers" :key="index">
      <input type="radio" v-model="answerSelected" :value="answer" :disabled="buttonSubmited" />
      <label v-html="answer"></label> <br />
    </div>
    <button @click="checkAnswer()" class="send" type="button" v-if="!buttonSubmited">
      En avant !
    </button>
    <span v-if="buttonSubmited">
      <h4 v-if="userSelectedIncorrectAnswer">
        ❌Vous avez sélecté la mauvaise réponse, la bonne était: {{ correctAnswer }}
      </h4>
      <h4 v-if="userSelectedCorrectAnswer">✅Vous avez sélecté la bonne réponse</h4>
      <button @click="apiCall()" class="send" type="button">Question suivante</button>
    </span>
  </div>
</template>

<script>
// librairies
import axios from "axios";

//components
import ScoreBoard from './components/ScoreBoard.vue';

export default {
  name: "App",
  components: {
    ScoreBoard
  },
  data() {
    return {
      question: undefined,
      correctAnswer: undefined,
      incorrectAnswers: undefined,
      answerSelected: undefined,
      buttonSubmited: false,
      userSelectedCorrectAnswer: false,
      userSelectedIncorrectAnswer: false,
      winCount:0,
      loseCount:0,
    };
  },
  computed: {
    answers() {
      let answers = JSON.parse(JSON.stringify(this.incorrectAnswers));
      answers.splice(Math.round(Math.random() * answers.length), 0, this.correctAnswer);
      return answers;
    },
  },
  methods: {
    checkAnswer() {
      if (!this.answerSelected) {
        alert("Veuillez choisir une réponse");
        return;
      }
      this.buttonSubmited = true;
      if (this.answerSelected) {
        if (this.correctAnswer === this.answerSelected) {
          // alert("Vous avez trouvé la bonne réponse, trop fort sérieux 😉");
          this.userSelectedCorrectAnswer = true;
          this.userSelectedIncorrectAnswer = false;
          this.winCount++
          return;
        }
        if (this.correctAnswer !== this.answerSelected) {
          // alert("Aie pas ça zinédine !");
          this.userSelectedIncorrectAnswer = true;
          this.userSelectedCorrectAnswer = false;
          this.loseCount++
          return;
        }
      }
    },
    async apiCall() {
      this.question = undefined;
      this.buttonSubmited = false;
      this.answerSelected = false;
      await axios
        .get("https://opentdb.com/api.php?amount=10&category=15&difficulty=easy&type=multiple")
        .then((response) => {
          const { results } = response.data;
          this.question = results[0].question;
          this.correctAnswer = results[0].correct_answer;
          this.incorrectAnswers = results[0].incorrect_answers;
        });
    },
  },
  created() {
    this.apiCall();
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px auto;
  max-width: 960px;

  input[type="radio"] {
    margin: 12px 4px;
  }

  button.send {
    margin-top: 12px;
    height: 40px;
    min-width: 120px;
    padding: 0 16px;
    color: #fff;
    background-color: #1867c0;
    border: 1px solid #1867c0;
    cursor: pointer;
  }
}
</style>
