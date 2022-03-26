<!-- Template !-->
<template>

<div class="main">

    <div class="headerWrapper">
      
      <div class="questNbr" v-if="questions.length > 0 && questionNumber <= 20">
          {{ questionNumber }}/20
      </div>

      <a href="https://github.com/Wishk6/pouroucontre" target="_blanck">
        <div class="githubLogoCircle">
          <img src="../assets/githubLogo.png" alt="Logo Github" title="Github Project" class="githubLogo">
        </div>
      </a>

      <div v-if="questions.length <=0" class="information">
        <div class="informationText">VOS REPONSES SERONT ENREGISTREES <div>ANNONYMEMENT</div></div>
    </div>

    </div>

    <div class="titleWrapper" v-if="questions.length <= 0">

        <div class="title">POUR</div>
        <div class="title">OU</div>
        <div class="title">CONTRE</div>

    </div>

    <div class="subTitleWrapper" v-if="questions.length <= 0">
        <div class="subTitle">Êtes vous en accord avec la majorité ?</div>
    </div>
    

    

    <div v-if="questions.length <= 0" class="startButtonWrapper">

        <div  v-on:click="getQuestions()" class="startButton">
            <div class="startText">
                COMMENCER
            </div>
        </div>

    </div>
 

        <div v-if="questions.length > 0 && questionNumber <= 20" class="secondDiv">

            <div class="questionWrapper">

              <div class="question">
                {{ question.content }}
              </div>

            </div>

            <div class="choiceWrapper">

                <div v-on:click=" storeData(question.id, question.yes + 1, question.no, true)"
                v-bind:class=" { forAgainstButton: boolAnswer == 0, forAgainstButtonDisabled: boolAnswer == 1, forAgainstButtonClicked : boolAnswer == 2 } ">

                    <div v-if="boolAnswer == 2">
                        {{
                        Math.round(((question.yes + 1) / (question.yes + question.no + 1)) * 100)
                        }}%
                    </div>

                    <div style="font-weight: bold;">POUR</div>

                    <div v-if="boolAnswer == 2">
                      {{ question.yes + 1 }} Votes
                    </div>

                </div>

                <div :disabled="boolAnswer == 2" v-on:click=" storeData(question.id, question.yes, question.no + 1, false)"
                v-bind:class="{forAgainstButton: boolAnswer == 0, forAgainstButtonDisabled : boolAnswer == 2, forAgainstButtonClicked : boolAnswer == 1 }">

                    <div v-if="boolAnswer == 1">
                      {{
                        Math.round(((question.no + 1) / (question.yes + question.no + 1)) * 100)
                      }}%
                    </div>

                    <div style="font-weight: bold;">CONTRE</div>

                    <div v-if="boolAnswer == 1">
                      {{ question.no + 1}} Votes
                    </div>

                </div>

            </div>

        </div>

            <div  v-if="questions.length > 0 && questionNumber < 20 && !end" v-on:click="getRandomQuestion()">
                <div v-bind:class="{ 'nextButtonHidden' : !answered, 'nextButton' : answered }">
                  QUESTION SUIVANTE
                </div>
            </div>

            <div class="finalDivWrapper">

                <div v-if="end && !showStats" v-on:click="updateData()" class="resultButton">Voir les résultats</div>

                <div v-if="showStats" class="stats">
                    Sur vos 20 réponses, {{ medium }} sont en accord avec la majorité,
                    soit {{ Math.round((medium / 20) * 100) }}%.
                </div>
        
                <a v-if="showStats" class="resultButton" href="https://pouroucontre.cbym5024.odns.fr">
                  MENU
                </a>

            </div>

    </div>

</template>

<!-- Script !-->
<script lang='ts'>

import { defineComponent } from 'vue'

export default defineComponent({
  name: 'QuizzComponent',
  data () {
    return {
      questions: [] /* all questions */,
      answers: [{ id: 0, yes: 0, no: 0 }], // users's answers */
      end: false /* checks if it the end of the questionary */,
      questionNumber: 0 /* question nbr */,
      question: '' /* actual question on the screen */,
      medium: 0 /* number of answers equals to average answers */,
      showStats: false /* checks if user want to see stats */,
      boolAnswer: 0 /*  toggle css ForAgainstBtn, 1: against, 2: for  (check if better way to do this) */,
      answered : false  /* check if answered actual quest */ 
    }
  },
  methods: {

    getQuestions: async function () {
      const requestOptions = {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' }
      }
      fetch('https://pouroucontre.cbym5024.odns.fr/api/question', requestOptions)
        .then(async (response) => {
          const data = await response.json()
          this.questions = data
          this.getRandomQuestion()
          if (!response.ok) {
            const error = (data && data.message) || response.status
            return Promise.reject(error)
          }
        })
        .catch((error) => {
          console.error('Erreur lors de la requête', error)
        })
    },

    getRandomQuestion: function () {
      const actualRandInt = Math.floor(Math.random() * this.questions.length)
      this.questionNumber++
      this.answered = false
      this.boolAnswer = 0
      this.question = this.questions[actualRandInt]

      this.questions = this.questions.filter((actualQuest) => {
        return (
          actualQuest !== this.questions[actualRandInt]
        ) /* remove actual question from questions obj */
      })
    },

    storeData: function (id: number, yes: number, no: number, choice: boolean) {
      this.answered = true
      if (this.questionNumber === 20) {
        this.end = true
      }
      /* store data in array to analyse it */
      if (this.boolAnswer === 0) {
        if (!choice) {
          const averageAnswer = no - 1 - yes >= 0
          this.boolAnswer = 1

          if (averageAnswer) { // if response same as average answer
            this.medium++
          }
        } else {
          const averageAnswer = yes - 1 - no >= 0
          this.boolAnswer = 2

          if (averageAnswer) {
            this.medium++
          }
        }
        const arr = { id: id, yes: yes, no: no }
        this.answers.push(arr)
      }
    },

    updateData: async function () {
      /* update data in Db */
      this.showStats = true
      this.answers.forEach(
        (element: { id: number; yes: number; no: number }) => {
          const requestOptions = {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              where: { id: element.id },
              update: {
                yes: element.yes,
                no: element.no
              }
            })
          }
          fetch('https://pouroucontre.cbym5024.odns.fr/api/question/update', requestOptions)
            .then(async (response) => {
              const data = await response.json()
              if (!response.ok) {
                const error = (data && data.message) || response.status
                return Promise.reject(error)
              }
            })
            .catch((error) => {
              console.error(
                'Erreur durant la mise à jour de la base de données',
                error
              )
            })
        })
    }
  } // close methods
})
</script>
<!-- Style !-->
<style scoped>
 @import 'quizzcomponent.css';
</style>
