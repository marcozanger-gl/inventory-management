<template>
  <div class="app">
    <AppSidebar
      :collapsed="collapsed"
      :open="drawerOpen"
      @toggle-collapse="toggleCollapsed"
      @close="closeDrawer"
      @show-profile-details="openProfileDetails"
      @show-tasks="openTasks"
    />
    <div
      v-if="drawerOpen"
      class="sidebar-overlay"
      aria-hidden="true"
      @click="closeDrawer"
    ></div>

    <!-- inert keeps focus and clicks inside the drawer while it is open -->
    <div class="app-body" :inert="drawerOpen">
      <header class="mobile-topbar">
        <button
          ref="hamburgerEl"
          type="button"
          class="hamburger"
          :aria-expanded="String(drawerOpen)"
          :aria-label="drawerOpen ? t('nav.closeMenu') : t('nav.openMenu')"
          @click="drawerOpen ? closeDrawer() : openDrawer()"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M4 6h16M4 12h16M4 18h16" />
          </svg>
        </button>
        <span class="mobile-topbar-title">{{ t('nav.companyName') }}</span>
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
import { ref, onMounted, onBeforeUnmount, computed, watch, nextTick } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import AppSidebar from './components/AppSidebar.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'

const SIDEBAR_KEY = 'inventory.sidebar.collapsed'

// localStorage can throw (private mode, blocked storage): fall back to no memory
const readCollapsed = () => {
  try {
    return window.localStorage.getItem(SIDEBAR_KEY) === '1'
  } catch (e) {
    return false
  }
}

const writeCollapsed = (value) => {
  try {
    window.localStorage.setItem(SIDEBAR_KEY, value ? '1' : '0')
  } catch (e) {
    // ignore: sidebar still works without persistence
  }
}

export default {
  name: 'App',
  components: {
    FilterBar,
    AppSidebar,
    ProfileDetailsModal,
    TasksModal
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const collapsed = ref(readCollapsed())
    const drawerOpen = ref(false)
    const hamburgerEl = ref(null)

    const toggleCollapsed = () => {
      collapsed.value = !collapsed.value
      writeCollapsed(collapsed.value)
    }

    const openDrawer = () => {
      drawerOpen.value = true
    }

    const closeDrawer = () => {
      if (!drawerOpen.value) return
      drawerOpen.value = false
      // Return focus to the hamburger once inert is lifted
      nextTick(() => {
        if (hamburgerEl.value) hamburgerEl.value.focus()
      })
    }

    // Close the drawer first so scroll lock and inert don't linger behind the modal
    const openProfileDetails = () => {
      closeDrawer()
      showProfileDetails.value = true
    }

    const openTasks = () => {
      closeDrawer()
      showTasks.value = true
    }

    const onKeydown = (e) => {
      if (e.key === 'Escape' && drawerOpen.value) closeDrawer()
    }

    // Lock page scroll while the drawer is open, restore on close
    let previousOverflow = ''
    watch(drawerOpen, (isOpen) => {
      if (isOpen) {
        previousOverflow = document.body.style.overflow
        document.body.style.overflow = 'hidden'
      } else {
        document.body.style.overflow = previousOverflow
      }
    })

    // The drawer only exists below 768px: close it when the viewport grows
    let mql = null
    const onMq = (e) => {
      if (e.matches && drawerOpen.value) {
        drawerOpen.value = false
      }
    }

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

    onMounted(() => {
      loadTasks()
      document.addEventListener('keydown', onKeydown)
      if (window.matchMedia) {
        mql = window.matchMedia('(min-width: 768px)')
        mql.addEventListener('change', onMq)
      }
    })

    onBeforeUnmount(() => {
      document.removeEventListener('keydown', onKeydown)
      if (mql) mql.removeEventListener('change', onMq)
      if (drawerOpen.value) document.body.style.overflow = previousOverflow
    })

    return {
      t,
      collapsed,
      drawerOpen,
      hamburgerEl,
      toggleCollapsed,
      openDrawer,
      closeDrawer,
      openProfileDetails,
      openTasks,
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
:root {
  /* Spacing, 4px base */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-7: 32px;
  --space-8: 48px;

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Colors: surfaces and text */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-text: #1e293b;
  --color-text-strong: #0f172a;
  --color-text-muted: #64748b;
  --color-border: #e2e8f0;
  --color-border-hover: #cbd5e1;
  --color-hover-bg: #f1f5f9;
  --color-accent: #2563eb;
  --color-accent-bg: #eff6ff;

  /* Colors: status */
  --color-success: #059669;
  --color-warning: #ea580c;
  --color-danger: #dc2626;
  --color-info: #2563eb;

  /* Sidebar */
  --sidebar-width: 240px;
  --sidebar-width-collapsed: 64px;
  --sidebar-bg: #0f172a;
  --sidebar-border: #1e293b;
  --sidebar-text: #cbd5e1;
  --sidebar-text-muted: #94a3b8;
  --sidebar-hover-bg: #1e293b;
  --sidebar-active-bg: #2563eb;
  --sidebar-active-text: #ffffff;
  --topbar-height: 56px;

  /* Typography */
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 0.938rem;
  --text-md: 1.125rem;
  --text-lg: 1.375rem;
  --text-xl: 1.875rem;
  --text-2xl: 2.25rem;
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --weight-bold: 700;
  --tracking-tight: -0.025em;

  /* Shadows */
  --shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.06);

  /* z-index scale: sticky content < sidebar < overlay < dropdown < modal.
     The open mobile drawer is the exception: it sits at overlay + 1. */
  --z-sticky: 90;
  --z-sidebar: 100;
  --z-overlay: 200;
  --z-dropdown: 300;
  --z-modal: 400;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-sans);
  background: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.app-body {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-width: 0;
}

.mobile-topbar {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  height: var(--topbar-height);
  padding: 0 var(--space-4);
  background: var(--color-surface);
  border-bottom: 1px solid var(--color-border);
  position: sticky;
  top: 0;
  z-index: var(--z-sticky);
}

.mobile-topbar-title {
  font-size: var(--text-base);
  font-weight: var(--weight-semibold);
  color: var(--color-text-strong);
  letter-spacing: var(--tracking-tight);
}

.hamburger {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  background: none;
  border: none;
  border-radius: var(--radius-md);
  color: var(--color-text-strong);
  cursor: pointer;
}

.hamburger:hover {
  background: var(--color-hover-bg);
}

.hamburger:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

.sidebar-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.5);
  z-index: var(--z-overlay);
}

.main-content {
  flex: 1;
  max-width: 1600px;
  width: 100%;
  min-width: 0;
  margin: 0 auto;
  padding: var(--space-6) var(--space-4);
}

@media (min-width: 768px) {
  .app {
    flex-direction: row;
  }

  .mobile-topbar,
  .sidebar-overlay {
    display: none;
  }

  .main-content {
    padding: var(--space-6) var(--space-7);
  }
}

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: var(--text-xl);
  font-weight: var(--weight-bold);
  color: var(--color-text-strong);
  margin-bottom: var(--space-2);
  letter-spacing: var(--tracking-tight);
}

.page-header p {
  color: var(--color-text-muted);
  font-size: var(--text-base);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: var(--color-border-hover);
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--color-text-muted);
  font-size: var(--text-sm);
  font-weight: var(--weight-semibold);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: var(--space-3);
}

.stat-value {
  font-size: var(--text-2xl);
  font-weight: var(--weight-bold);
  color: var(--color-text-strong);
  letter-spacing: var(--tracking-tight);
}

.stat-card.warning .stat-value {
  color: var(--color-warning);
}

.stat-card.success .stat-value {
  color: var(--color-success);
}

.stat-card.danger .stat-value {
  color: var(--color-danger);
}

.stat-card.info .stat-value {
  color: var(--color-accent);
}

.card {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  border: 1px solid var(--color-border);
  margin-bottom: var(--space-5);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-4);
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: var(--text-md);
  font-weight: var(--weight-bold);
  color: var(--color-text-strong);
  letter-spacing: var(--tracking-tight);
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-bg);
  border-top: 1px solid var(--color-border);
  border-bottom: 1px solid var(--color-border);
}

th {
  text-align: left;
  padding: var(--space-2) var(--space-3);
  font-weight: var(--weight-semibold);
  color: #475569;
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: var(--space-2) var(--space-3);
  border-top: 1px solid var(--color-hover-bg);
  color: #334155;
  font-size: var(--text-sm);
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--color-bg);
}

.badge {
  display: inline-block;
  padding: var(--space-1) var(--space-3);
  border-radius: var(--radius-sm);
  font-size: var(--text-xs);
  font-weight: var(--weight-semibold);
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
  padding: var(--space-8);
  color: var(--color-text-muted);
  font-size: var(--text-base);
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: var(--text-base);
}
</style>
