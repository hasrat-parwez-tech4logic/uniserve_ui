<script setup>
import { ref } from 'vue';
import AppSidebar from './components/organisms/AppSidebar.vue';
import DashboardView from './pages/DashboardView.vue';
import UserJourneyView from './pages/UserJourneyView.vue';

// Current view state
const currentView = ref('dashboard');

const handleNavigate = (view) => {
  currentView.value = view;
};
</script>

<template>
  <div class="flex h-screen overflow-hidden bg-gray-50">
    <!-- Sidebar (80px width) -->
    <div class="w-20 flex-shrink-0">
      <AppSidebar :currentView="currentView" @navigate="handleNavigate" />
    </div>
    
    <!-- Main Content -->
    <div class="flex-1 overflow-hidden">
      <DashboardView v-if="currentView === 'dashboard'" />
      <UserJourneyView v-else-if="currentView === 'users'" />
    </div>
  </div>
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
