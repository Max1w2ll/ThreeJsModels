<template>
  <MainPage />
</template>

<script lang="ts">
/* eslint-disable */
import { defineComponent } from "vue";
import MainPage from "./components/Main.vue";

export default defineComponent({
  name: "App",
  components: {
    MainPage,
  },
});
</script>

<style>
body {
  margin: 0;
  overflow-y: hidden;
  overflow-x: hidden;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
