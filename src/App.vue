<script setup>
import { RouterLink, RouterView, useRoute } from 'vue-router';
import SpecialDays from './components/Title.vue';
import { computed, ref, onMounted } from 'vue';

const route = useRoute();
const routeKey = computed(() => route.fullPath);
const url = window.location.pathname;

const iframeLoaded = ref(false);
const startIrisOut = ref(false);

const onIframeLoad = () => {
  
  setTimeout(() => {
    startIrisOut.value = true;

    // Then wait for the iris animation to complete before removing loader
    setTimeout(() => {
      iframeLoaded.value = true;
    }, 2000); // match CSS transition time
  }, 2000);
  
};
</script>

<template>
  <SpecialDays v-if="url != '/one'"></SpecialDays>
  <header>
    <div class="wrapper">
      
      <router-view v-slot="{ Component }">
      <Transition name="bounce" mode="out-in">
        <component :is="Component" :key="routeKey"/>
      </Transition>
    </router-view>
    </div>
  </header>

  <!-- Loader Cover -->
  <div v-if="!iframeLoaded && url != '/one'" :class="['loader-cover', { 'iris-out': startIrisOut }]">
    <div class="spinner"></div>
    <p v-if="!startIrisOut">Loading... Wait a sec</p>
  </div>

  <iframe v-if="url != '/one'" id="cake" src='https://my.spline.design/cakeproduction-hGBj5X11umLdG6ufU34QDZ3Q/' frameborder='0' width='80%' height='100%' @load="onIframeLoad"></iframe>
  <svg v-if="url != '/one'" preserveAspectRatio="xMidYMid slice" id="svg" viewBox="10 10 80 80">
  <defs>
  <pattern id="img1" patternUnits="userSpaceOnUse" width="100" height="100">
    <image src="download.jpg" x="0" y="0" width="100" height="100"/>
  </pattern>
  </defs>
    <path fill="#9b5de5" class="out-top" d="M37-5C25.1-14.7,5.7-19.1-9.2-10-28.5,1.8-32.7,31.1-19.8,49c15.5,21.5,52.6,22,67.2,2.3C59.4,35,53.7,8.5,37-5Z"/>
    <path fill="#f15bb5" class="in-top" d="M20.6,4.1C11.6,1.5-1.9,2.5-8,11.2-16.3,23.1-8.2,45.6,7.4,50S42.1,38.9,41,24.5C40.2,14.1,29.4,6.6,20.6,4.1Z"/>
    <path fill="#00bbf9" class="out-bottom" d="M105.9,48.6c-12.4-8.2-29.3-4.8-39.4.8-23.4,12.8-37.7,51.9-19.1,74.1s63.9,15.3,76-5.6c7.6-13.3,1.8-31.1-2.3-43.8C117.6,63.3,114.7,54.3,105.9,48.6Z"/>
    <path fill="#00f5d4" class="in-bottom" d="M102,67.1c-9.6-6.1-22-3.1-29.5,2-15.4,10.7-19.6,37.5-7.6,47.8s35.9,3.9,44.5-12.5C115.5,92.6,113.9,74.6,102,67.1Z"/>

  </svg>



  
</template>


<style scoped>
canvas > a {
  display: none;
}

.loader-cover {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: white;
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  clip-path: circle(150% at 50% 50%);
  transition: clip-path 1s ease-in-out;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #9b5de5;
  border-top: 5px solid transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}

.loader-cover.iris-out {
  clip-path: circle(0% at 50% 50%);
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}
</style>
