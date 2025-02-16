<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue';
</script>

<template>
  <header>
    <h1>BPM Counter</h1>
  </header>
  <main></main>
  <footer></footer>
</template>

<style scoped>
header {
  width: 100%;
  height: 50px;
  background-color: #444;
  display: flex;
  align-items: center;

  h1 {
    margin-left: 10px;
    color: #eee;
  }
}

main {
  height: calc(100vh - 100px);
}

footer {
  width: 100%;
  height: 50px;
  background-color: #222;
}
</style>
