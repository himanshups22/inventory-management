<template>
  <div class="app" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <aside class="sidebar">
      <!-- Logo section -->
      <div class="sidebar-logo">
        <template v-if="!sidebarCollapsed">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="subtitle">{{ t('nav.subtitle') }}</span>
        </template>
        <span v-else class="logo-abbr">CC</span>
      </div>

      <!-- Nav links with SVG icons -->
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/></svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M16.5 9.4l-9-5.19M21 16V8a2 2 0 00-1-1.73l-7-4a2 2 0 00-2 0l-7 4A2 2 0 003 8v8a2 2 0 001 1.73l7 4a2 2 0 002 0l7-4A2 2 0 0021 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M16 4h2a2 2 0 012 2v14a2 2 0 01-2 2H6a2 2 0 01-2-2V6a2 2 0 012-2h2"/><rect x="8" y="2" width="8" height="4" rx="1" ry="1"/></svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6"/></svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/restocking" :class="{ active: $route.path === '/restocking' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0114.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0020.49 15"/></svg>
          <span class="nav-label">Restocking</span>
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
          <span class="nav-label">Reports</span>
        </router-link>
      </nav>

      <!-- Bottom: language + profile + collapse toggle -->
      <div class="sidebar-bottom">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
        <button class="collapse-btn" @click="toggleSidebar">
          <template v-if="!sidebarCollapsed">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="11 17 6 12 11 7"/><polyline points="18 17 13 12 18 7"/></svg>
          </template>
          <template v-else>
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="13 17 18 12 13 7"/><polyline points="6 17 11 12 6 7"/></svg>
          </template>
        </button>
      </div>
    </aside>

    <div class="main-area">
      <div class="top-bar">
        <h2 class="page-title">{{ currentPageTitle }}</h2>
        <div class="top-bar-spacer"></div>
        <FilterBar />
      </div>
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
    const $route = useRoute()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const sidebarCollapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')

    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
      localStorage.setItem('sidebar-collapsed', sidebarCollapsed.value)
    }

    const currentPageTitle = computed(() => {
      const routeMap = {
        '/': t('nav.overview'),
        '/inventory': t('nav.inventory'),
        '/orders': t('nav.orders'),
        '/spending': t('nav.finance'),
        '/demand': t('nav.demandForecast'),
        '/restocking': 'Restocking',
        '/reports': 'Reports'
      }
      return routeMap[$route.path] || ''
    })

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
      $route,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed,
      toggleSidebar,
      currentPageTitle
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--bg);
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

:root {
  --sidebar-width: 260px;
  --sidebar-collapsed-width: 64px;
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-text-hover: #f1f5f9;
  --sidebar-active-bg: rgba(59, 130, 246, 0.1);
  --sidebar-active-text: #3b82f6;
  --sidebar-border: #1e293b;
  --bg: #f1f5f9;
  --surface: #ffffff;
  --surface-border: #e2e8f0;
  --surface-hover-border: #cbd5e1;
  --elevated-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.06);
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #64748b;
  --text-faint: #94a3b8;
  --accent: #3b82f6;
  --accent-light: #eff6ff;
  --accent-text: #1e40af;
  --success: #059669;
  --success-bg: #d1fae5;
  --success-text: #065f46;
  --warning: #ea580c;
  --warning-bg: #fed7aa;
  --warning-text: #92400e;
  --danger: #dc2626;
  --danger-bg: #fecaca;
  --danger-text: #991b1b;
  --info: #2563eb;
  --info-bg: #dbeafe;
  --info-text: #1e40af;
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 14px;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 0.938rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.375rem;
  --font-size-2xl: 1.875rem;
  --top-bar-height: 56px;
}

/* ===== APP LAYOUT ===== */

.app {
  display: flex;
  min-height: 100vh;
}

/* ===== SIDEBAR ===== */

.sidebar {
  width: var(--sidebar-width);
  min-width: var(--sidebar-width);
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
  background: var(--sidebar-bg);
  display: flex;
  flex-direction: column;
  z-index: 200;
  transition: width 0.2s ease, min-width 0.2s ease;
  overflow: hidden;
}

.app.sidebar-collapsed .sidebar {
  width: var(--sidebar-collapsed-width);
  min-width: var(--sidebar-collapsed-width);
}

.sidebar-logo {
  padding: var(--space-6) var(--space-5);
  border-bottom: 1px solid var(--sidebar-border);
  min-height: 73px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.app.sidebar-collapsed .sidebar-logo {
  align-items: center;
  padding: var(--space-6) var(--space-3);
}

.sidebar-logo h1 {
  font-size: var(--font-size-xl);
  font-weight: 700;
  color: #ffffff;
  white-space: nowrap;
  letter-spacing: -0.025em;
}

.sidebar-logo .subtitle {
  font-size: var(--font-size-xs);
  color: var(--sidebar-text);
  white-space: nowrap;
  border: none;
  padding: 0;
  margin-top: 2px;
}

.logo-abbr {
  font-size: 1.25rem;
  font-weight: 700;
  color: #ffffff;
  text-align: center;
  display: block;
}

.sidebar-nav {
  flex: 1;
  padding: var(--space-3) 0;
  overflow-y: auto;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-5);
  color: var(--sidebar-text);
  text-decoration: none;
  font-weight: 500;
  font-size: var(--font-size-sm);
  transition: all 0.15s ease;
  border-left: 3px solid transparent;
  white-space: nowrap;
}

.app.sidebar-collapsed .sidebar-nav a {
  justify-content: center;
  padding: var(--space-3);
}

.sidebar-nav a:hover {
  color: var(--sidebar-text-hover);
  background: rgba(255, 255, 255, 0.04);
}

.sidebar-nav a.active {
  color: var(--sidebar-active-text);
  background: var(--sidebar-active-bg);
  border-left-color: var(--sidebar-active-text);
}

.sidebar-nav a svg {
  flex-shrink: 0;
}

.nav-label {
  opacity: 1;
  transition: opacity 0.15s ease;
}

.app.sidebar-collapsed .nav-label {
  opacity: 0;
  width: 0;
  overflow: hidden;
}

.sidebar-bottom {
  padding: var(--space-3) var(--space-5);
  border-top: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
}

.app.sidebar-collapsed .sidebar-bottom {
  padding: var(--space-3);
  align-items: center;
}

.collapse-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  width: 100%;
  padding: var(--space-2) var(--space-3);
  background: transparent;
  border: 1px solid var(--sidebar-border);
  border-radius: var(--radius-sm);
  color: var(--sidebar-text);
  cursor: pointer;
  font-size: var(--font-size-xs);
  transition: all 0.15s ease;
}

.collapse-btn:hover {
  color: var(--sidebar-text-hover);
  background: rgba(255, 255, 255, 0.04);
}

/* ===== MAIN AREA ===== */

.main-area {
  flex: 1;
  margin-left: var(--sidebar-width);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  transition: margin-left 0.2s ease;
}

.app.sidebar-collapsed .main-area {
  margin-left: var(--sidebar-collapsed-width);
}

.top-bar {
  height: var(--top-bar-height);
  background: var(--surface);
  border-bottom: 1px solid var(--surface-border);
  display: flex;
  align-items: center;
  padding: 0 var(--space-8);
  position: sticky;
  top: 0;
  z-index: 100;
}

.page-title {
  font-size: var(--font-size-lg);
  font-weight: 700;
  color: var(--text-primary);
  white-space: nowrap;
  letter-spacing: -0.025em;
}

.top-bar-spacer {
  flex: 1;
}

.main-content {
  flex: 1;
  background: var(--bg);
  padding: var(--space-6) var(--space-8);
}

/* ===== PAGE HEADER ===== */

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: var(--font-size-2xl);
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--text-muted);
  font-size: var(--font-size-base);
}

/* ===== STATS GRID ===== */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--surface);
  padding: 1.25rem;
  border-radius: var(--radius-md);
  border: 1px solid var(--surface-border);
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: var(--surface-hover-border);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: var(--text-muted);
  font-size: var(--font-size-sm);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: var(--warning);
}

.stat-card.success .stat-value {
  color: var(--success);
}

.stat-card.danger .stat-value {
  color: var(--danger);
}

.stat-card.info .stat-value {
  color: var(--info);
}

/* ===== CARD ===== */

.card {
  background: var(--surface);
  border-radius: var(--radius-md);
  padding: 1.25rem;
  border: 1px solid var(--surface-border);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--surface-border);
}

.card-title {
  font-size: var(--font-size-lg);
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.025em;
}

/* ===== TABLE ===== */

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid var(--surface-border);
  border-bottom: 1px solid var(--surface-border);
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: var(--text-secondary);
  font-size: var(--font-size-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: var(--font-size-sm);
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

/* ===== BADGES ===== */

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: var(--radius-sm);
  font-size: var(--font-size-xs);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: var(--success-bg);
  color: var(--success-text);
}

.badge.warning {
  background: var(--warning-bg);
  color: var(--warning-text);
}

.badge.danger {
  background: var(--danger-bg);
  color: var(--danger-text);
}

.badge.info {
  background: var(--info-bg);
  color: var(--info-text);
}

.badge.increasing {
  background: var(--success-bg);
  color: var(--success-text);
}

.badge.decreasing {
  background: var(--danger-bg);
  color: var(--danger-text);
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: var(--danger-bg);
  color: var(--danger-text);
}

.badge.medium {
  background: var(--warning-bg);
  color: var(--warning-text);
}

.badge.low {
  background: var(--info-bg);
  color: var(--info-text);
}

/* ===== LOADING / ERROR ===== */

.loading {
  text-align: center;
  padding: 3rem;
  color: var(--text-muted);
  font-size: var(--font-size-base);
}

.error {
  background: #fef2f2;
  border: 1px solid var(--danger-bg);
  color: var(--danger-text);
  padding: var(--space-4);
  border-radius: 8px;
  margin: var(--space-4) 0;
  font-size: var(--font-size-base);
}
</style>
