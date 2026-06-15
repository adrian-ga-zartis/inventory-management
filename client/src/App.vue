<template>
  <div class="app">
    <aside class="sidebar">
      <!-- Logo section -->
      <div class="sidebar-logo">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
      </div>

      <!-- Nav links -->
      <nav class="sidebar-nav">
        <router-link to="/">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/></svg>
          {{ t('nav.overview') }}
        </router-link>
        <router-link to="/inventory">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
          {{ t('nav.inventory') }}
        </router-link>
        <router-link to="/orders">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>
          {{ t('nav.orders') }}
        </router-link>
        <router-link to="/spending">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
          {{ t('nav.finance') }}
        </router-link>
        <router-link to="/demand">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>
          {{ t('nav.demandForecast') }}
        </router-link>
        <router-link to="/restocking">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></svg>
          {{ t('nav.restocking') }}
        </router-link>
        <router-link to="/reports">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
          Reports
        </router-link>
      </nav>

      <!-- Filters section -->
      <div class="sidebar-filters">
        <FilterBar />
      </div>

      <!-- User profile pinned to bottom -->
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu @show-profile-details="showProfileDetails = true" @show-tasks="showTasks = true" />
      </div>
    </aside>

    <div class="main-wrapper">
      <!-- Topbar -->
      <header class="topbar">
        <h2 class="topbar-title">{{ currentPageTitle }}</h2>
      </header>

      <!-- Scrollable content -->
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

// Map route paths to display names for the topbar title
const PAGE_TITLES = {
  '/': 'Overview',
  '/inventory': 'Inventory',
  '/orders': 'Orders',
  '/spending': 'Finance',
  '/demand': 'Demand Forecast',
  '/restocking': 'Restocking',
  '/reports': 'Reports'
}

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const route = useRoute()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Derive topbar title from current route path
    const currentPageTitle = computed(() => PAGE_TITLES[route.path] || 'Dashboard')

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      currentPageTitle,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }

:root {
  --bg: #0f172a;
  --surface: #1e293b;
  --border: #334155;
  --text: #e2e8f0;
  --subtle: #94a3b8;
  --accent: #38bdf8;
  --sidebar-width: 240px;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: var(--bg);
  color: var(--text);
  -webkit-font-smoothing: antialiased;
}

/* ── Layout shell ── */
.app { display: flex; min-height: 100vh; }

.sidebar {
  position: fixed;
  left: 0; top: 0;
  width: var(--sidebar-width);
  height: 100vh;
  background: var(--bg);
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  z-index: 100;
  overflow-y: auto;
}

.main-wrapper {
  margin-left: var(--sidebar-width);
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: var(--bg);
}

/* ── Sidebar logo ── */
.sidebar-logo {
  padding: 1.25rem 1rem 1rem;
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}

.sidebar-logo h1 {
  font-size: 1rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.02em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-subtitle {
  font-size: 0.7rem;
  color: var(--subtle);
  display: block;
  margin-top: 0.2rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* ── Sidebar nav ── */
.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 1rem;
  margin: 0 0.5rem;
  border-radius: 6px;
  color: var(--subtle);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: all 0.15s ease;
  white-space: nowrap;
}

.sidebar-nav a:hover {
  color: var(--text);
  background: rgba(255,255,255,0.06);
}

.sidebar-nav a.router-link-active,
.sidebar-nav a.router-link-exact-active {
  color: var(--accent);
  background: rgba(56, 189, 248, 0.1);
  border-left: 3px solid var(--accent);
  padding-left: calc(1rem - 3px);
}

.sidebar-nav a svg {
  flex-shrink: 0;
  opacity: 0.7;
}

.sidebar-nav a:hover svg,
.sidebar-nav a.router-link-active svg {
  opacity: 1;
}

/* ── Sidebar filters section ── */
.sidebar-filters {
  padding: 0.75rem 0.5rem;
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
  flex-shrink: 0;
}

/* ── Sidebar footer ── */
.sidebar-footer {
  padding: 0.75rem 1rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-shrink: 0;
}

/* ── Topbar ── */
.topbar {
  height: 56px;
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  padding: 0 1.5rem;
  position: sticky;
  top: 0;
  z-index: 50;
  flex-shrink: 0;
}

.topbar-title {
  font-size: 1rem;
  font-weight: 600;
  color: var(--text);
  letter-spacing: -0.01em;
}

/* ── Main content ── */
.main-content {
  flex: 1;
  padding: 1.5rem;
  overflow-y: auto;
}

/* ── Page header ── */
.page-header { margin-bottom: 1.5rem; }

.page-header h2 {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p { color: var(--subtle); font-size: 0.9rem; }

/* ── Stats grid ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.stat-card {
  background: var(--surface);
  padding: 1.25rem;
  border-radius: 10px;
  border: 1px solid var(--border);
  transition: border-color 0.2s;
}

.stat-card:hover { border-color: #475569; }

.stat-label {
  color: var(--subtle);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value { color: #fb923c; }
.stat-card.success .stat-value { color: #34d399; }
.stat-card.danger  .stat-value { color: #f87171; }
.stat-card.info    .stat-value { color: #60a5fa; }

/* ── Cards ── */
.card {
  background: var(--surface);
  border-radius: 10px;
  padding: 1.25rem;
  border: 1px solid var(--border);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--border);
}

.card-title {
  font-size: 1rem;
  font-weight: 600;
  color: var(--text);
  letter-spacing: -0.01em;
}

/* ── Tables ── */
.table-container { overflow-x: auto; }

table { width: 100%; border-collapse: collapse; }

thead {
  background: rgba(255,255,255,0.03);
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: var(--subtle);
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid var(--border);
  color: #cbd5e1;
  font-size: 0.875rem;
}

tbody tr { transition: background 0.15s ease; }
tbody tr:hover { background: rgba(255,255,255,0.03); }

/* ── Badges ── */
.badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  border-radius: 5px;
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.badge.success   { background: rgba(52,211,153,0.15);  color: #34d399; }
.badge.warning   { background: rgba(251,146,60,0.15);  color: #fb923c; }
.badge.danger    { background: rgba(248,113,113,0.15); color: #f87171; }
.badge.info      { background: rgba(96,165,250,0.15);  color: #60a5fa; }
.badge.increasing{ background: rgba(52,211,153,0.15);  color: #34d399; }
.badge.decreasing{ background: rgba(248,113,113,0.15); color: #f87171; }
.badge.stable    { background: rgba(167,139,250,0.15); color: #a78bfa; }
.badge.high      { background: rgba(248,113,113,0.15); color: #f87171; }
.badge.medium    { background: rgba(251,146,60,0.15);  color: #fb923c; }
.badge.low       { background: rgba(96,165,250,0.15);  color: #60a5fa; }

/* ── States ── */
.loading { text-align: center; padding: 3rem; color: var(--subtle); }

.error {
  background: rgba(248,113,113,0.1);
  border: 1px solid rgba(248,113,113,0.3);
  color: #f87171;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
}
</style>
