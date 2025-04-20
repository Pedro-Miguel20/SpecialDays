<script setup>
import Rocker from '../components/Rocker.vue';
import { ref } from 'vue';

const childData = ref('');

const handleUpdate = (newVal) => {
  childData.value = newVal;
  console.log(childData)
};

</script>

<template>
<div class="rockerWrapper"><div class="mymessage" v-show="!childData"><p>Oi, essa é minha humilde surpresa para você, me dediquei muito para que ficasse legal, mas não está perfeito, mas funciona. Feliz aniversário, te desejo o melhor. Logo, para comemorar, nada melhor que tocarmos um parabéns.</p>Teclas do game:<div class="keys"><p style="color: lightgreen;">A</p><p style="color: crimson;">S</p><p style="color: gold;">D</p><p>&nbsp</p><p style="color: cornflowerblue;">K</p><p style="color: tomato;">L</p></div></div><Rocker @update = 'handleUpdate'></Rocker></div></template>

<style>

@font-face {
        font-family: 'keys';
        src: url('../Fonts/212Keyboard.otf');
    }

.rockerWrapper{
  width: 100%;
  height: inherit;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: white;
  flex-direction: column;
}

.mymessage{
  width: 500px;
  text-align: center;
  color: darkslategrey;
  font-size: 24px;
}

.keys{
  display: flex;
  justify-content: center;
}
.keys p{
  font-family: 'keys';
  font-size: 84px;
  margin: 5px;
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