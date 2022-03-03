<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import { onMounted } from 'vue';
import VueBinaryViwer from './components/VueBinaryViwer.vue'

onMounted(() => {
  window.addEventListener('beforeunload', () => {
    return confirm("确定退出吗");
  })
})
</script>

<template>
  <VueBinaryViwer />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: linear-gradient(135deg, rgb(20, 216, 230), rgb(98, 240, 63));
  height: 100vh;
  overflow: auto;
}
</style>
