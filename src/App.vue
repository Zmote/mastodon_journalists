<template>
  <img alt="Mastodon logo" class="app-logo" src="./assets/logo.png">
  <JournalistList msg="Welcome to Your Vue.js App"/>
</template>

<script>
import JournalistList from "@/components/JournalistList";

export default {
  name: 'App',
  components: {
    JournalistList
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 1rem;
}

.app-logo{
  width: 100px;
  height: auto;
}
</style>
