<template>
  <div class="app" :class="{ collapsed: sidebarCollapsed }">
    <aside class="sidebar">
      <div class="logo">
        <div class="logo-mark" aria-hidden="true">{{ t('nav.companyName').charAt(0) }}</div>
        <div class="logo-text">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>
      <nav class="nav-tabs" aria-label="Main">
          <router-link to="/" :class="{ active: $route.path === '/' }" :title="t('nav.overview')">
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/></svg>
            <span class="nav-label">{{ t('nav.overview') }}</span>
          </router-link>
          <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }" :title="t('nav.inventory')">
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 8l-9-5-9 5v8l9 5 9-5z"/><path d="M3 8l9 5 9-5M12 13v8"/></svg>
            <span class="nav-label">{{ t('nav.inventory') }}</span>
          </router-link>
          <router-link to="/orders" :class="{ active: $route.path === '/orders' }" :title="t('nav.orders')">
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/><rect x="9" y="3" width="6" height="4" rx="1"/><path d="M9 12h6M9 16h6"/></svg>
            <span class="nav-label">{{ t('nav.orders') }}</span>
          </router-link>
          <router-link to="/spending" :class="{ active: $route.path === '/spending' }" :title="t('nav.finance')">
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M12 2v20M17 6H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
            <span class="nav-label">{{ t('nav.finance') }}</span>
          </router-link>
          <router-link to="/demand" :class="{ active: $route.path === '/demand' }" :title="t('nav.demandForecast')">
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M3 17l6-6 4 4 8-8"/><path d="M15 7h6v6"/></svg>
            <span class="nav-label">{{ t('nav.demandForecast') }}</span>
          </router-link>
          <router-link to="/reports" :class="{ active: $route.path === '/reports' }" :title="t('nav.reports')">
            <svg class="nav-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M3 3v18h18"/><path d="M7 15v-4M12 15V8M17 15v-6"/></svg>
            <span class="nav-label">{{ t('nav.reports') }}</span>
          </router-link>
      </nav>
      <button
        class="sidebar-toggle"
        type="button"
        :aria-expanded="!sidebarCollapsed"
        :aria-label="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        @click="toggleSidebar"
      >
        <svg class="nav-icon toggle-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <path d="M11 17l-5-5 5-5M18 17l-5-5 5-5"/>
        </svg>
        <span class="nav-label">Collapse</span>
      </button>
    </aside>

    <div class="content">
      <header class="topbar">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </header>
      <FilterBar />
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
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Sidebar collapse: remembered choice wins, otherwise collapse on narrow screens
    const narrowQuery = window.matchMedia('(max-width: 900px)')
    let stored = null
    try { stored = localStorage.getItem('sidebarCollapsed') } catch (e) { /* storage unavailable */ }
    const sidebarCollapsed = ref(stored === null ? narrowQuery.matches : stored === 'true')

    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
      try { localStorage.setItem('sidebarCollapsed', String(sidebarCollapsed.value)) } catch (e) { /* ignore */ }
    }

    // Crossing the breakpoint resets to the screen-size default
    narrowQuery.addEventListener('change', (e) => { sidebarCollapsed.value = e.matches })

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
      showProfileDetails,
      showTasks,
      tasks,
      sidebarCollapsed,
      toggleSidebar,
      addTask,
      deleteTask,
      toggleTask
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
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Design tokens (see .claude/skills/frontend-design/SKILL.md) */
:root {
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-text-active: #ffffff;
  --sidebar-active-bg: #1e293b;
  --accent: #3b82f6;
  --bg: #f8fafc;
  --surface: #ffffff;
  --border: #e2e8f0;
  --text: #0f172a;
  --text-muted: #64748b;
  --radius: 10px;
  --shadow: 0 1px 2px rgba(15, 23, 42, 0.06);
  --sidebar-width: 240px;
  --topbar-height: 56px;
}

/* Layout: fixed sidebar on the left, scrolling content on the right */
.app {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: var(--sidebar-width);
  flex-shrink: 0;
  background: var(--sidebar-bg);
  color: var(--sidebar-text);
  position: sticky;
  top: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 1.25rem 0.75rem;
  gap: 1.5rem;
  z-index: 110;
  overflow-y: auto;
}

.logo {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0 0.5rem;
}

.logo-mark {
  width: 36px;
  height: 36px;
  flex-shrink: 0;
  border-radius: 8px;
  background: var(--accent);
  color: #fff;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo-text {
  min-width: 0;
}

.logo h1 {
  font-size: 0.938rem;
  font-weight: 700;
  color: var(--sidebar-text-active);
  letter-spacing: -0.01em;
  line-height: 1.2;
}

.subtitle {
  display: block;
  font-size: 0.75rem;
  color: var(--sidebar-text);
  margin-top: 2px;
}

.nav-tabs {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.nav-tabs a {
  position: relative;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  height: 40px;
  padding: 0 0.75rem;
  color: var(--sidebar-text);
  text-decoration: none;
  font-weight: 500;
  font-size: 0.875rem;
  border-radius: 8px;
  transition: background-color 0.15s ease, color 0.15s ease;
}

.nav-tabs a:hover {
  color: var(--sidebar-text-active);
  background: var(--sidebar-active-bg);
}

.nav-tabs a:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.nav-tabs a.active {
  color: var(--sidebar-text-active);
  background: var(--sidebar-active-bg);
}

/* 3px accent bar on the active item */
.nav-tabs a.active::before {
  content: '';
  position: absolute;
  left: -0.75rem;
  top: 8px;
  bottom: 8px;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: var(--accent);
}

.nav-icon {
  flex-shrink: 0;
}

.content {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.topbar {
  height: var(--topbar-height);
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 1rem;
  padding: 0 2rem;
  position: sticky;
  top: 0;
  z-index: 100;
}

.main-content {
  flex: 1;
  max-width: 1400px;
  width: 100%;
  margin: 0 auto;
  padding: 1.5rem 2rem 2rem;
}

/* Let grid children shrink so wide charts/tables scroll inside their card
   instead of stretching the whole page now that the sidebar takes width */
.main-content .charts-grid > *,
.main-content .stats-grid > * {
  min-width: 0;
}

.sidebar {
  transition: width 0.2s ease;
}

.sidebar-toggle {
  margin-top: auto;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  height: 40px;
  padding: 0 0.75rem;
  background: transparent;
  border: none;
  border-radius: 8px;
  color: var(--sidebar-text);
  font: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
}

.sidebar-toggle:hover {
  color: var(--sidebar-text-active);
  background: var(--sidebar-active-bg);
}

.sidebar-toggle:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.toggle-icon {
  transition: transform 0.2s ease;
}

/* Collapsed: 64px icons-only rail (set by the toggle, or by default on narrow screens) */
.app.collapsed .sidebar {
  width: 64px;
  padding: 1rem 0.5rem;
  align-items: center;
}

.app.collapsed .logo {
  padding: 0;
}

.app.collapsed .logo-text,
.app.collapsed .nav-label {
  display: none;
}

.app.collapsed .nav-tabs a,
.app.collapsed .sidebar-toggle {
  justify-content: center;
  width: 44px;
  padding: 0;
}

.app.collapsed .nav-tabs a.active::before {
  left: -0.5rem;
}

.app.collapsed .toggle-icon {
  transform: rotate(180deg);
}

@media (max-width: 900px) {
  .topbar {
    padding: 0 1rem;
  }

  .main-content {
    padding: 1rem;
  }
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #64748b;
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: var(--surface);
  padding: 1.25rem;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  box-shadow: var(--shadow);
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

.card {
  background: var(--surface);
  border-radius: var(--radius);
  padding: 1.25rem;
  border: 1px solid var(--border);
  box-shadow: var(--shadow);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
