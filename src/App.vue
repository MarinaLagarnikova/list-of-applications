<template>
  <StartScreen v-if="!selectedMode" @select="selectedMode = $event" />
  <MortgagePage v-else :mode="selectedMode" />
</template>

<script setup>
import { ref } from 'vue'
import StartScreen from './components/StartScreen.vue'
import MortgagePage from '../MortgagePage.vue'

const selectedMode = ref(null)
</script>
