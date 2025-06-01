<template>
  <div class="about">
    <h1>Bem vindo {{ nome }}</h1>
  </div>
</template>


<script setup lang="ts">
import { ref } from 'vue';

const nome = ref("Nome");
</script>
<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
