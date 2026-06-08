<template>
  <!-- <img alt="誠揚智能科技 logo" class="logo" src="./assets/誠揚智能科技_logo_無字.png"> -->
  <HelloWorld msg="誠揚智能科技"/>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.logo {
  width: 200px;
}
</style>
