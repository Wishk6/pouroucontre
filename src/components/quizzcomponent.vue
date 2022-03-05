<template>

<div class="main">

    <div class="githubLogoWrapper">
      <div class="githubLogoCircle">
        <img src="../assets/githubLogo.png" alt="Logo Github" class="githubLogo">
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

    <div class="startButtonWrapper">

        <div v-if="questions.length <= 0" v-on:click="getQuestions()" class="startButton">
            <div class="startText">
                COMMENCER
            </div>
        </div>

    </div>

        <div v-if="questions.length > 0 && questionNumber <= 20" class="secondDiv">

            <div class="question">
              {{ question.content }}
               <div class="questNbr">
              {{ questionNumber }}/20
            </div>
            </div>

            <div class="choiceWrapper">

                <div :disabled="clicked" v-on:click=" storeData(question.id, question.yes + 1, question.no, true)"
                    class="forButton">

                    <div>Pour</div>
                    <div v-if="clicked">{{ question.yes }}</div>

                    <div v-if="clicked">
                        {{
                        Math.round((question.yes / (question.yes + question.no)) * 100)
                        }}%
                    </div>
                </div>

                <div :disabled="clicked" v-on:click=" storeData(question.id, question.yes, question.no + 1, false)"
                    class="againstButton">

                    <div>Contre</div>
                    <div v-if="clicked">
                        {{ question.no }}
                    </div>

                    <div v-if="clicked">
                        {{
                        Math.round((question.no / (question.yes + question.no)) * 100)
                        }}%
                    </div>

                </div>

            </div>

            <div>

                <div v-if="questionNumber <= 20 && ! end" v-on:click="getRandomQuestion()" class="nextButton">
                    <div>Question suivante</div>
                </div>

            </div>

            <div>

                <div v-if="end" v-on:click="updateData()">Voir les résultats</div>
                <div v-if="showStats">
                    Sur vos 20 réponses, {{ medium }} sont en accord avec la majorité,
                    soit {{ Math.round((medium / 20) * 100) }}%
                </div>

            </div>

            <div v-if="showStats">
                <div>Recommencer</div>
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
      clicked: false /* handle affichage des moyennes */,
      end: false /* checks if it the end of the questionary */,
      questionNumber: 0 /* actual question nbr */,
      question: '' /* actual question on the screen */,
      medium: 0 /* number of answers equals to average answers */,
      showStats: false /* checks if user want to see stats */
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
      this.clicked = false
      this.question = this.questions[actualRandInt]

      if (this.questionNumber === 20) {
        this.end = true
      }
      this.questions = this.questions.filter((actualQuest) => {
        return (
          actualQuest !== this.questions[actualRandInt]
        ) /* remove actual question from questions obj */
      })
    },

    storeData: function (id: number, yes: number, no: number, choice: boolean) {
      /* store data in array to analyse it */
      if (!choice) {
        const averageAnswer = no - 1 - yes >= 0
        if (averageAnswer) {
          this.medium++
        }
      } else {
        const averageAnswer = yes - 1 - no >= 0
        if (averageAnswer) {
          this.medium++
        }
      }
      const arr = { id: id, yes: yes, no: no }
      this.answers.push(arr)
      this.clicked = true
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

<style scoped>

/* Title */
.titleWrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-weight: bold;
}

.title {
  font-size: 70px;
}

.subTitleWrapper {
  display: flex;
  margin-top:20vh;
  flex-direction: column;
  align-items: center;
}

.subTitle::after{
  content: "";
  display: block;
  width:100%;
  padding-top: 3px;
  border-bottom: 2px solid var(--globalColor);
}

/* Github Link */
.githubLogoWrapper {
  display: flex;
  justify-content: flex-end;
  padding:25px;
}

.githubLogoCircle {
  display: flex;
  border-radius: 100%;
  background: white;
  width: 80px;
  height: 80px;
  justify-content: center;
  align-items: center;
}

.githubLogo {
  width:64px;
  height:64px;
}

/* Start Button */
.startButtonWrapper {
  display:flex;
  justify-content: center;
  margin:40px;
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

.startButton:hover {
  transform: scale(1.05);
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

.question {
 display: flex;
 border : solid 3px var(--elemBorder);
 background: var(--elemBg);
 max-width: 30vw;
}

.questNbr {
display: block;
border:solid 1px var(--globalColor);
border-radius: 100%;
width:64px;
height:64px;
}

/* Choices */
.choiceWrapper {
  display: flex;
  gap:15px;
}

.forButton {
  cursor: pointer;
  border: solid 1px var(--globalColor);
}

.againstButton {
  cursor: pointer;
  border: solid 1px var(--globalColor);
}

/* Next question */
.nextButton {
cursor:pointer;
}
</style>
