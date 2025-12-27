<script setup>
import { LayoutDashboard, Users } from 'lucide-vue-next';

const props = defineProps({
  currentView: {
    type: String,
    required: true
  }
});

const emit = defineEmits(['navigate']);

const navItems = [
  { id: 'dashboard', label: 'Dashboard', icon: LayoutDashboard },
  { id: 'users', label: 'Users', icon: Users }
];
</script>

<template>
  <div class="flex flex-col h-full bg-gradient-to-b from-gray-900 to-gray-800 border-r border-gray-700">
    <!-- Logo/Brand -->
    <div class="p-4 border-b border-gray-700">
      <div class="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-lg flex items-center justify-center">
        <span class="text-white font-bold text-xl">U</span>
      </div>
    </div>
    
    <!-- Navigation Items -->
    <nav class="flex-1 py-4">
      <button
        v-for="item in navItems"
        :key="item.id"
        @click="emit('navigate', item.id)"
        :class="[
          'w-full flex flex-col items-center justify-center py-4 px-2 transition-all duration-200 relative group',
          currentView === item.id
            ? 'text-white bg-blue-600/20'
            : 'text-gray-400 hover:text-white hover:bg-gray-700/50'
        ]"
        :title="item.label"
      >
        <!-- Active indicator -->
        <div 
          v-if="currentView === item.id"
          class="absolute left-0 top-0 bottom-0 w-1 bg-blue-500 rounded-r"
        ></div>
        
        <component :is="item.icon" :size="24" class="mb-1" />
        <span class="text-xs font-medium">{{ item.label }}</span>
      </button>
    </nav>
    
    <!-- Footer -->
    <div class="p-4 border-t border-gray-700 text-center">
      <p class="text-xs text-gray-500">UniServe</p>
    </div>
  </div>
</template>

<style scoped>
button:focus {
  outline: none;
}
</style>
