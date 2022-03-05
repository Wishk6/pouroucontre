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

@import url('https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@100&display=swap');

 * {
    font-size:28px;
    margin: 0;
    padding: 0;
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
--globalColor : white;
--agreeColor : green;
--disagreeeColor: red;
}

</style>
