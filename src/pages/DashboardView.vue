<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { dashboardAPI } from '../services/api';
import StatCard from '../components/molecules/StatCard.vue';
import { 
  Users, 
  Activity, 
  MessageSquare, 
  Clock, 
  TrendingUp,
  RefreshCw,
  UserPlus,
  Zap
} from 'lucide-vue-next';

// State
const stats = ref(null);
const userGrowth = ref([]);
const sessionTrends = ref([]);
const hourlyActivity = ref([]);
const topFlowSteps = ref([]);
const realTimeStats = ref(null);
const loading = ref(false);
const error = ref(null);
const selectedPeriod = ref(7);

// Auto-refresh interval
let refreshInterval = null;

// Fetch all dashboard data
const fetchDashboardData = async () => {
  try {
    loading.value = true;
    error.value = null;

    const [statsRes, growthRes, trendsRes, activityRes, stepsRes, realtimeRes] = await Promise.all([
      dashboardAPI.getStats(selectedPeriod.value),
      dashboardAPI.getUserGrowth(30),
      dashboardAPI.getSessionTrends(30),
      dashboardAPI.getHourlyActivity(7),
      dashboardAPI.getTopFlowSteps(5),
      dashboardAPI.getRealTimeStats()
    ]);

    stats.value = statsRes.data;
    userGrowth.value = growthRes.data;
    sessionTrends.value = trendsRes.data;
    hourlyActivity.value = activityRes.data;
    topFlowSteps.value = stepsRes.data;
    realTimeStats.value = realtimeRes.data;
  } catch (err) {
    console.error('Error fetching dashboard data:', err);
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

// Format duration in seconds to readable format
const formatDuration = (seconds) => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return mins > 0 ? `${mins}m ${secs}s` : `${secs}s`;
};

// Calculate user growth percentage
const userGrowthPercentage = computed(() => {
  if (!stats.value) return null;
  const { totalUsers, newUsers } = stats.value.overview;
  if (totalUsers === 0) return null;
  const percentage = ((newUsers / totalUsers) * 100).toFixed(1);
  return { value: `${percentage}%`, direction: 'up' };
});

// Calculate session growth
const sessionGrowthPercentage = computed(() => {
  if (!stats.value) return null;
  const { totalSessions, sessionsInPeriod } = stats.value.overview;
  if (totalSessions === 0) return null;
  const percentage = ((sessionsInPeriod / totalSessions) * 100).toFixed(1);
  return { value: `${percentage}%`, direction: 'up' };
});

// Get peak activity hour
const peakHour = computed(() => {
  if (!hourlyActivity.value.length) return 'N/A';
  const peak = hourlyActivity.value.reduce((max, item) => 
    item.count > max.count ? item : max
  , { count: 0, hour: 0 });
  return `${peak.hour}:00`;
});

// Lifecycle
onMounted(() => {
  fetchDashboardData();
  // Auto-refresh every 30 seconds
  refreshInterval = setInterval(fetchDashboardData, 30000);
});

onUnmounted(() => {
  if (refreshInterval) {
    clearInterval(refreshInterval);
  }
});
</script>

<template>
  <div class="flex flex-col h-full bg-gray-50">
    <!-- Header -->
    <div class="bg-white border-b border-gray-200 px-8 py-6">
      <div class="flex items-center justify-between">
        <div>
          <h1 class="text-3xl font-bold text-gray-900">Dashboard</h1>
          <p class="text-gray-500 mt-1">Monitor your UniServe analytics and performance</p>
        </div>
        
        <div class="flex items-center gap-4">
          <!-- Period Selector -->
          <select 
            v-model="selectedPeriod"
            @change="fetchDashboardData"
            class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option :value="7">Last 7 days</option>
            <option :value="14">Last 14 days</option>
            <option :value="30">Last 30 days</option>
          </select>
          
          <!-- Refresh Button -->
          <button
            @click="fetchDashboardData"
            :disabled="loading"
            class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors disabled:opacity-50 flex items-center gap-2"
          >
            <RefreshCw :size="18" :class="{ 'animate-spin': loading }" />
            Refresh
          </button>
        </div>
      </div>
    </div>

    <!-- Content -->
    <div class="flex-1 overflow-y-auto p-8">
      <!-- Loading State -->
      <div v-if="loading && !stats" class="flex items-center justify-center h-full">
        <div class="text-center">
          <RefreshCw :size="48" class="animate-spin text-blue-500 mx-auto mb-4" />
          <p class="text-gray-600">Loading dashboard...</p>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="bg-red-50 border border-red-200 rounded-lg p-6 text-center">
        <p class="text-red-600">{{ error }}</p>
        <button @click="fetchDashboardData" class="mt-4 px-4 py-2 bg-red-600 text-white rounded-lg">
          Try Again
        </button>
      </div>

      <!-- Dashboard Content -->
      <div v-else-if="stats" class="space-y-8">
        <!-- Real-time Stats Banner -->
        <div v-if="realTimeStats" class="bg-gradient-to-r from-blue-600 to-purple-600 rounded-xl p-6 text-white shadow-lg">
          <div class="flex items-center gap-2 mb-4">
            <Zap :size="20" />
            <h2 class="text-lg font-semibold">Real-time Activity (Last 5 minutes)</h2>
          </div>
          <div class="grid grid-cols-3 gap-6">
            <div>
              <p class="text-blue-100 text-sm mb-1">Active Users</p>
              <p class="text-3xl font-bold">{{ realTimeStats.activeUsers }}</p>
            </div>
            <div>
              <p class="text-blue-100 text-sm mb-1">New Sessions</p>
              <p class="text-3xl font-bold">{{ realTimeStats.recentSessions }}</p>
            </div>
            <div>
              <p class="text-blue-100 text-sm mb-1">Messages</p>
              <p class="text-3xl font-bold">{{ realTimeStats.recentMessages }}</p>
            </div>
          </div>
        </div>

        <!-- Key Metrics -->
        <div>
          <h2 class="text-xl font-bold text-gray-900 mb-4">Overview</h2>
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
            <StatCard
              title="Total Users"
              :value="stats.overview.totalUsers"
              :icon="Users"
              color="blue"
              :trend="userGrowthPercentage ? { ...userGrowthPercentage, label: 'new users' } : null"
            />
            
            <StatCard
              title="Total Sessions"
              :value="stats.overview.totalSessions"
              :icon="Activity"
              color="green"
              :trend="sessionGrowthPercentage ? { ...sessionGrowthPercentage, label: 'this period' } : null"
            />
            
            <StatCard
              title="Active Sessions"
              :value="stats.overview.activeSessions"
              :icon="TrendingUp"
              color="purple"
            />
            
            <StatCard
              title="Total Messages"
              :value="stats.overview.totalMessages"
              :icon="MessageSquare"
              color="amber"
            />
          </div>
        </div>

        <!-- Secondary Metrics -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div class="bg-white rounded-xl p-6 shadow-sm border border-gray-100">
            <div class="flex items-center gap-3 mb-4">
              <div class="p-2 bg-cyan-100 rounded-lg">
                <Clock :size="20" class="text-cyan-600" />
              </div>
              <h3 class="font-semibold text-gray-900">Avg Session Duration</h3>
            </div>
            <p class="text-2xl font-bold text-gray-900">{{ formatDuration(stats.overview.avgSessionDuration) }}</p>
          </div>

          <div class="bg-white rounded-xl p-6 shadow-sm border border-gray-100">
            <div class="flex items-center gap-3 mb-4">
              <div class="p-2 bg-rose-100 rounded-lg">
                <UserPlus :size="20" class="text-rose-600" />
              </div>
              <h3 class="font-semibold text-gray-900">New Users</h3>
            </div>
            <p class="text-2xl font-bold text-gray-900">{{ stats.overview.newUsers }}</p>
            <p class="text-sm text-gray-500 mt-1">Last {{ selectedPeriod }} days</p>
          </div>

          <div class="bg-white rounded-xl p-6 shadow-sm border border-gray-100">
            <div class="flex items-center gap-3 mb-4">
              <div class="p-2 bg-purple-100 rounded-lg">
                <Activity :size="20" class="text-purple-600" />
              </div>
              <h3 class="font-semibold text-gray-900">Peak Hour</h3>
            </div>
            <p class="text-2xl font-bold text-gray-900">{{ peakHour }}</p>
            <p class="text-sm text-gray-500 mt-1">Most active time</p>
          </div>
        </div>

        <!-- Charts Row -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
          <!-- User Types Distribution -->
          <div class="bg-white rounded-xl p-6 shadow-sm border border-gray-100">
            <h3 class="text-lg font-semibold text-gray-900 mb-4">User Types</h3>
            <div class="space-y-3">
              <div v-for="(count, type) in stats.distributions.userTypes" :key="type" class="flex items-center justify-between">
                <div class="flex items-center gap-3">
                  <div 
                    :class="[
                      'w-3 h-3 rounded-full',
                      type === 'ACTIVE' ? 'bg-green-500' : 
                      type === 'RETURNING' ? 'bg-blue-500' : 
                      type === 'NEW' ? 'bg-purple-500' : 'bg-gray-400'
                    ]"
                  ></div>
                  <span class="text-gray-700 font-medium">{{ type }}</span>
                </div>
                <span class="text-2xl font-bold text-gray-900">{{ count }}</span>
              </div>
            </div>
          </div>

          <!-- Session Sources -->
          <div class="bg-white rounded-xl p-6 shadow-sm border border-gray-100">
            <h3 class="text-lg font-semibold text-gray-900 mb-4">Session Sources</h3>
            <div class="space-y-3">
              <div v-for="(count, source) in stats.distributions.sessionSources" :key="source" class="flex items-center justify-between">
                <div class="flex items-center gap-3">
                  <div 
                    :class="[
                      'w-3 h-3 rounded-full',
                      source === 'QR_CODE' ? 'bg-blue-500' : 
                      source === 'DIRECT_LINK' ? 'bg-green-500' : 
                      source === 'AD_CLICK' ? 'bg-amber-500' : 
                      source === 'ORGANIC' ? 'bg-purple-500' : 'bg-gray-400'
                    ]"
                  ></div>
                  <span class="text-gray-700 font-medium">{{ source.replace('_', ' ') }}</span>
                </div>
                <span class="text-2xl font-bold text-gray-900">{{ count }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Top Flow Steps -->
        <div v-if="topFlowSteps.length > 0" class="bg-white rounded-xl p-6 shadow-sm border border-gray-100">
          <h3 class="text-lg font-semibold text-gray-900 mb-4">Top Journey Steps</h3>
          <div class="space-y-2">
            <div v-for="(step, index) in topFlowSteps" :key="step.step" class="flex items-center gap-4">
              <div class="w-8 h-8 rounded-full bg-blue-100 text-blue-600 flex items-center justify-center font-bold text-sm">
                {{ index + 1 }}
              </div>
              <div class="flex-1">
                <p class="font-medium text-gray-900">{{ step.step }}</p>
              </div>
              <div class="text-right">
                <p class="text-lg font-bold text-gray-900">{{ step.count }}</p>
                <p class="text-xs text-gray-500">visits</p>
              </div>
            </div>
          </div>
        </div>
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
