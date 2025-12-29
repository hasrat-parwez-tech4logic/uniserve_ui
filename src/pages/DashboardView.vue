<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { useAuthStore } from '../stores/auth';
import { useOrganizationStore } from '../stores/organization';
import OrganizationCard from '../components/molecules/OrganizationCard.vue';
import { 
  Building2, 
  Plus,
  RefreshCw,
  Search
} from 'lucide-vue-next';

// Stores
const router = useRouter();
const authStore = useAuthStore();
const organizationStore = useOrganizationStore();

// State
const loading = ref(false);
const error = ref(null);
const searchQuery = ref('');

// Filtered organizations based on search
const filteredOrganizations = computed(() => {
  if (!searchQuery.value) {
    return organizationStore.organizations;
  }
  return organizationStore.organizations.filter(org =>
    org.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    (org.description && org.description.toLowerCase().includes(searchQuery.value.toLowerCase()))
  );
});

// Fetch organizations
const fetchOrganizations = async () => {
  try {
    loading.value = true;
    error.value = null;
    await organizationStore.fetchOrganizations();
  } catch (err) {
    console.error('Error fetching organizations:', err);
    error.value = err.message || 'Failed to load organizations';
  } finally {
    loading.value = false;
  }
};

// Handle organization card click
const handleOrganizationClick = (organization) => {
  router.push(`/organizations/${organization._id}/projects`);
};

// Lifecycle
onMounted(() => {
  fetchOrganizations();
});
</script>

<template>
  <div class="flex flex-col h-full bg-gray-50">
    <!-- Header -->
    <div class="bg-white border-b border-gray-200 px-6 py-5">
      <div class="flex items-center justify-between mb-3">
        <div>
          <h1 class="text-2xl font-bold text-gray-900">Organizations</h1>
          <p class="text-gray-500 mt-1 text-sm">Select an organization to view its projects</p>
        </div>
        
        <div class="flex items-center gap-4">
          <!-- Refresh Button -->
          <button
            @click="fetchOrganizations"
            :disabled="loading"
            class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors disabled:opacity-50 flex items-center gap-2"
          >
            <RefreshCw :size="18" :class="{ 'animate-spin': loading }" />
            Refresh
          </button>

          <!-- Add Organization Button (Super Admin only) -->
          <button
            v-if="authStore.isSuperAdmin"
            class="px-4 py-2 bg-gradient-to-r from-blue-600 to-purple-600 text-white rounded-lg hover:shadow-lg transition-all flex items-center gap-2"
          >
            <Plus :size="18" />
            New Organization
          </button>
        </div>
      </div>

      <!-- Search Bar -->
      <div class="relative">
        <Search :size="20" class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400" />
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search organizations..."
          class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
      </div>
    </div>

    <!-- Content -->
    <div class="flex-1 overflow-y-auto p-6">
      <!-- Loading State -->
      <div v-if="loading && !organizationStore.organizations.length" class="flex items-center justify-center h-full">
        <div class="text-center">
          <RefreshCw :size="48" class="animate-spin text-blue-500 mx-auto mb-4" />
          <p class="text-gray-600">Loading organizations...</p>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="bg-red-50 border border-red-200 rounded-lg p-6 text-center">
        <p class="text-red-600">{{ error }}</p>
        <button @click="fetchOrganizations" class="mt-4 px-4 py-2 bg-red-600 text-white rounded-lg">
          Try Again
        </button>
      </div>

      <!-- No Organizations -->
      <div v-else-if="filteredOrganizations.length === 0" class="flex items-center justify-center h-full">
        <div class="text-center">
          <Building2 :size="64" class="text-gray-300 mx-auto mb-4" />
          <p class="text-gray-600 text-lg mb-2">
            {{ searchQuery ? 'No organizations found' : 'No organizations yet' }}
          </p>
          <p class="text-gray-500 text-sm mb-4">
            {{ searchQuery ? 'Try a different search term' : 'Create your first organization to get started' }}
          </p>
          <button 
            v-if="!searchQuery && authStore.isSuperAdmin"
            class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors inline-flex items-center gap-2"
          >
            <Plus :size="18" />
            Create Organization
          </button>
        </div>
      </div>

      <!-- Organizations Grid -->
      <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <OrganizationCard
          v-for="organization in filteredOrganizations"
          :key="organization._id"
          :organization="organization"
          @click="handleOrganizationClick"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.overflow-y-auto::-webkit-scrollbar {
  width: 8px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: #f1f5f9;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}
</style>
