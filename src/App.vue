<script setup lang="ts">
import DynamicLogo from "./components/DynamicLogo.vue";
</script>

<template>
  <div class="bg">
    <DynamicLogo />
  </div>
</template>

<style scoped>
.bg {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  /* background-color: rgb(116, 116, 116); */
  background-color: #000;
}
</style>
