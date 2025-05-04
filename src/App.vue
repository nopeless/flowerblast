<script setup lang="ts">
import BlockBlast from './components/BlockBlast.vue';
</script>

<template>
  <div style="max-width: min(50vh, 80vw); margin-left: auto; margin-right: auto;">
    <BlockBlast></BlockBlast>
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
}

html {
  background-color: var(--background);
  height: 100dvh;
}

* {
  color: var(--text);
}
</style>
