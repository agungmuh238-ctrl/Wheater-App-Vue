
<script setup>
import { ref, onMounted } from 'vue'
import SplashScreen from './components/splashScreen.vue'
import WeatherApp from './components/theWeather.vue'

const isSplashVisible = ref(true)

onMounted(() => {
  setTimeout(() => {
    isSplashVisible.value = false
  }, 2500)
})
</script>


<template>
  <Transition name="fade" mode="out-in">
    <SplashScreen
      v-if="isSplashVisible"
      key="splash"
    />

    <WeatherApp
      v-else
      key="main"
    />
  </Transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

