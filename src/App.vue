<template>
  <quizz-component/>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import quizzComponent from './components/quizzcomponent.vue'

export default defineComponent({
  name: 'App',
  components: {
    quizzComponent
  }
})
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@300&display=swap');

@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300&display=swap');

 * {
    font-size:28px;
    margin: 0;
    padding: 0;
}

a {
  text-decoration: none;
  font-weight: bold;
  font-family: 'Roboto';
  color : var(--globalColor)
}

html {
  font-family: 'Roboto Slab', serif;
  background: url(assets/mainBg.png) no-repeat center fixed;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  color:var(--globalColor);
}

:root {
--globalColor : #ffffff;
--shadow : #00000040;
--globalBg1: #452a7a;
--globalBg2: #713f81;
--linear : linear-gradient(-80deg, var(--globalBg1), var(--globalBg2));
}

</style>
