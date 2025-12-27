<script setup>
import UserJourneyView from './pages/UserJourneyView.vue'
</script>

<template>
  <UserJourneyView />
</template>

<style>
/* Ensure full viewport height */
#app {
  @apply h-screen overflow-hidden;
}

/* Global styles */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
