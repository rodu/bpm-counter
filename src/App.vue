<script setup lang="ts">
import Tapper from './components/Tapper.vue';
</script>

<template>
  <header>
    <h1>BPM Counter</h1>
  </header>
  <main>
    <Tapper />
  </main>
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
  display: flex;
  align-items: center;
  justify-content: center;
  height: calc(100vh - 100px);
}

footer {
  width: 100%;
  height: 50px;
  background-color: #222;
}
</style>
