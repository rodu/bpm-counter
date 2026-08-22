<script setup lang="ts">
import Tapper from './components/Tapper.vue';
</script>

<template>
  <header>
    <h1>
      BPM Counter
      <small>by Robbiesse</small>
    </h1>
  </header>
  <main>
    <Tapper />
  </main>
  <footer>
    <a
      href="https://www.mixcloud.com/robbiesse/"
      target="_blank"
      rel="noopener noreferrer"
    >
      Robbiesse on Mixcloud
    </a>
  </footer>
</template>

<style scoped>
header {
  width: 100%;
  height: 60px;
  background-color: #173b4d;
  display: flex;
  align-items: center;

  h1 {
    margin-left: 10px;
    color: #c3cab0;

    small {
      font-size: 0.5em;
    }
  }
}

main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: calc(100vh - 100px);
}

footer {
  width: 100%;
  height: 50px;
  background-color: #0f2938;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding: 0 16px;
  box-sizing: border-box;

  a {
    color: #aab8bd;
    font-family: 'Space Grotesk', 'Avenir Next', sans-serif;
    font-size: 0.875rem;
    font-weight: 600;
  }

  a:hover,
  a:focus-visible {
    color: #c3cab0;
  }
}
</style>
