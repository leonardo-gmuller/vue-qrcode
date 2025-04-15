<script setup>
import QrGenerator from './components/QrGenerator.vue';
</script>

<template>
    <h1>Gerador de QR Code</h1>
  <QrGenerator />
  <h4>Criado por Leonardo Müller</h4>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
