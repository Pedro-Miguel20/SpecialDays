<script setup>
import Rocker from '../components/Rocker.vue';
</script>

<template><div class="rockerWrapper"><Rocker></Rocker></div></template>

<style>
.rockerWrapper{
  width: 100%;
  height: inherit;
  display: flex;
  justify-content: center;
  align-items: end;
  background-color: darkslategrey;
}

header{
    height: 100%;
}

.wrapper{
    justify-content: center;
    height: 100%;
    align-items: end;
}
</style>