<!-- Template !-->
<template>

<div class="main">

    <div class="headerWrapper">

            <div class="questNbr" v-if="questions.length > 0 && questionNumber <= 20">
                {{ questionNumber }}/20
              </div>

      <div class="githubLogoCircle">
        <img src="../assets/githubLogo.png" alt="Logo Github" title="Github Project" class="githubLogo">
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

                <div v-if="questions.length > 0 && questionNumber < 20 && !end" v-on:click="getRandomQuestion()">
                    <div class="nextButton">
                      QUESTION SUIVANTE
                    </div>
                </div>

            <div class="finalDivWrapper">

                <div v-if="end && !showStats" v-on:click="updateData()" class="resultButton">Voir les résultats</div>

                <div v-if="showStats">
                    Sur vos 20 réponses, {{ medium }} sont en accord avec la majorité,
                    soit {{ Math.round((medium / 20) * 100) }}%
                </div>

                <div v-if="showStats" class="resultButton">
                    <a href="http://localhost:8081/">MENU</a>
                </div>

            </div>

    </div>

</template>

<!-- Script !-->
<script lang='ts'>

import { defineComponent } from 'vue'

export default defineComponent({
  name: 'quizzComponent',
  data () {
    return {
      questions: [] /* all questions */,
      answers: [{ id: 0, yes: 0, no: 0 }], // users's answers */
      end: false /* checks if it the end of the questionary */,
      questionNumber: 0 /* question nbr */,
      question: '' /* actual question on the screen */,
      medium: 0 /* number of answers equals to average answers */,
      showStats: false /* checks if user want to see stats */,
      boolAnswer: 0 /*  toggle css ForAgainstBtn, 1: against, 2: for  (check if better way to do this) */
    }
  },
  methods: {

    getQuestions: async function () {
      const requestOptions = {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' }
      }
      fetch('http://localhost:3000/api/question', requestOptions)
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
      this.boolAnswer = 0
      this.question = this.questions[actualRandInt]

      this.questions = this.questions.filter((actualQuest) => {
        return (
          actualQuest !== this.questions[actualRandInt]
        ) /* remove actual question from questions obj */
      })
    },

    storeData: function (id: number, yes: number, no: number, choice: boolean) {
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
      console.log(this.answers)
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
          fetch('http://localhost:3000/api/question/update', requestOptions)
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
        }
      )
    }
  } // close methods
})
</script>
<!-- Style !-->
<style scoped>

/* Title */
.titleWrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top:-50px;
}

.title {
  font-size: 70px;
  font-weight: bold;
  filter:drop-shadow(0px 4px 3px rgb(0 0 0 / 25%));
}

.subTitleWrapper {
  display: flex;
  margin-top:28vh;
  flex-direction: column;
  align-items: center;
}

.subTitle::after {
  content: "";
  display: block;
  animation: width 0.3s ease;
  width:100%;
  padding-top: 3px;
  border-bottom: 1px solid var(--globalColor);
  box-shadow: 0px 4px 6px 0px rgb(0 0 0 / 20%);
}

/* Header */

.questNbr {
  display: flex;
  border: solid 4px var(--globalColor);
  border-radius: 100%;
  width: 72px;
  font-size:22px;
  height: 72px;
  justify-content: center;
  align-items: center;
}

.headerWrapper {
  display: flex;
  justify-content:space-between;
  padding: 45px 45px 0px 45px;
}

.githubLogoCircle {
  display: flex;
  border-radius: 100%;
  background: white;
  width: 80px;
  height: 80px;
  justify-content: center;
  align-items: center;
  transition: transform 0.2s ease-in, box-shadow 0.3s ease;
}

.githubLogoCircle:hover {
  cursor: pointer;
  transform: scale(1.1);
  box-shadow: 1px 5px 4px rgb(0 0 0 / 20%);
  transition: transform 0.2s ease-in, box-shadow 0.3s ease;
}

.githubLogo {
  width:64px;
  height:64px;
}

/* Start Button */
.startButtonWrapper {
  display:flex;
  justify-content: center;
  margin:50px;
}

.startButton {
  cursor: pointer;
  display:flex;
  justify-content: center;
  align-items: center;
  width:245px;
  height:70px;
  border: solid 2px var(--globalColor);
  transition: transform 0.3s ease;
}

.startButton:before {
  content: " ";
  display: block;
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  inset: 0 0 0 0;
  background: var(--linear);
  z-index: -1;
  transition: transform .3s ease;
}

.startButton::before {
  transform: scaleX(0);
  transform-origin: bottom left;
}

.startButton:hover::before {
  transform: scaleX(1);
  transform-origin: bottom left;
}

.startButton:hover {
  transition: transform 0.2s ease-out;
  transform: scale(1.07);
}

.startText {
  font-family: 'Roboto', sans-serif;
  font-weight: bold;
}

/* Questions */
.secondDiv {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.questionWrapper {
  display: flex;
  justify-content: center;
  gap:20px;
  transition: all 0.6s ease;
}

.question {
  content:'';
  border: solid 1px var(--globalColor);
  border-radius: 5px;
  max-width: 60%;
  font-size: 40px;
  padding: 20px 40px 20px 40px;
  box-shadow: rgba(14, 30, 37, 0.12) 0px 2px 4px 0px, rgba(14, 30, 37, 0.32) 0px 2px 16px 0px;
  animation: cubic-bezier(0.075, 0.82, 0.165, 1);
}

/* Choices */
.choiceWrapper {
  display: flex;
  gap: 20vw;
  margin: 17vh 0 12vh 0;
  align-items: center;
}

.forAgainstButton {
  min-width: 11vw;
  height:auto;
  cursor: pointer;
  border: solid 1px var(--globalColor);
  border-radius: 5px;
  padding: 10px 20px;
  display: flex;
  flex-direction: column;
  font-family: 'Roboto', sans-serif;
  transition: transform 0.3s ease, min-width 0.3s ease, height 0.3s ease;
}

.forAgainstButton:hover {
  transform: scale(1.15);
}

.forAgainstButtonClicked {
  margin-top:-6%;
  min-width: 25vw;
  height:auto;
  background: rgba(128, 85, 161, 0.6);
  border: solid 1px var(--globalColor);
  border-radius: 5px;
  padding: 10px 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  font-family: 'Roboto', sans-serif;
  transition: min-width 0.3s ease;
}

.forAgainstButtonDisabled {
  min-width: 109px;
  border: solid 1px var(--globalColor);
  border-radius: 5px;
  padding: 10px 20px;
  display: flex;
  cursor:no-drop;
  flex-direction: column;
  font-family: 'Roboto', sans-serif;
}

/* Next question */

.nextButton {
  position: absolute;
  left:0;
  right:0;
  bottom: 100px;
  margin-left: auto;
  margin-right: auto;
  width:280px;
  text-align: center;
  cursor:pointer;
  border: solid 1px var(--globalColor);
  font-family: 'Roboto', sans-serif;
  font-weight: bold;
  padding: 20px;
  transition : transform 0.2s ease,background 0.3s ease, color 0.3s ease;
}

.nextButton:before {
  content: " ";
  display: block;
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  inset: 0 0 0 0;
  background: var(--globalColor);
  z-index: -1;
  transition: transform .5s ease, border 0.5s ease;
  transform: scaleX(0);
  color: var(--globalBg2);
  transform-origin: bottom right;
}

.nextButton:hover {
  color : var(--globalBg2);
}

.nextButton:hover::before {
  transform: scaleX(1);
  transform-origin: bottom left;
}

.nextButton:active {
  transform: scale(0.98);
  border:none;
}

.finalDivWrapper {
  display:flex;
  flex-direction: column;
  justify-content: center;
  gap:60px;
  margin-top:10px;
  align-items: center;
}

.resultButton {
  cursor: pointer;
  border: solid 1px var(--globalColor);
  padding: 10px 20px;
  text-align: center;
  border-radius: 50px;
  transition : all 0.3s ease;
}

.resultButton:hover {
  background:var(--globalColor);
  color : var(--globalBg1);
}

.resultButton:active {
  transform:scale(0.97);
}

a:hover {
  color : var(--globalBg1);
}

@media screen and (max-width:650px) {
/*menu screen*/
  * {
    font-size: 4vw;
  }

  .titleWrapper {
    margin-top: unset;
  }

  /*question screen */
  .secondDiv {
    margin-top:1rem;
  }

  .forAgainstButton {
    min-width: 17vw;
  }
}
</style>
