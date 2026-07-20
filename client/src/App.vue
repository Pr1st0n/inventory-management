<template>
  <div
    class="app-shell"
    :class="{
      'is-collapsed': collapsed && !isMobile,
      'is-mobile-open': mobileOpen && isMobile
    }"
  >
    <aside class="sidebar" :class="{ 'is-collapsed': collapsed && !isMobile }">
      <div class="sidebar__brand">
        <span class="sidebar__logo">
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M3 7l9-4 9 4-9 4-9-4z" stroke="currentColor" stroke-width="1.7" stroke-linejoin="round"/>
            <path d="M3 12l9 4 9-4" stroke="currentColor" stroke-width="1.7" stroke-linejoin="round"/>
            <path d="M3 17l9 4 9-4" stroke="currentColor" stroke-width="1.7" stroke-linejoin="round"/>
          </svg>
        </span>
        <span class="sidebar__brand-name">
          <b>{{ t('nav.companyName') }}</b>
          <span>{{ t('nav.subtitle') }}</span>
        </span>
      </div>

      <button
        class="nav-item sidebar__toggle"
        type="button"
        @click="toggleSidebar"
        :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        aria-label="Toggle sidebar"
      >
        <svg class="sidebar__toggle-icon" viewBox="0 0 24 24" fill="none" aria-hidden="true">
          <path d="M15 6l-6 6 6 6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        <span>{{ collapsed ? 'Expand' : 'Collapse' }}</span>
      </button>

      <nav class="nav-group">
        <div class="nav-label">Operations</div>

        <router-link
          to="/"
          class="nav-item"
          active-class="nav-item"
          exact-active-class="router-link-active"
          :title="t('nav.overview')"
          @click="closeMobile"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <rect x="3" y="3" width="7" height="7" rx="1.5" stroke="currentColor"/>
            <rect x="14" y="3" width="7" height="7" rx="1.5" stroke="currentColor"/>
            <rect x="14" y="14" width="7" height="7" rx="1.5" stroke="currentColor"/>
            <rect x="3" y="14" width="7" height="7" rx="1.5" stroke="currentColor"/>
          </svg>
          <span>{{ t('nav.overview') }}</span>
        </router-link>

        <router-link
          to="/inventory"
          class="nav-item"
          :title="t('nav.inventory')"
          @click="closeMobile"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M21 8l-9-5-9 5v8l9 5 9-5V8z" stroke="currentColor" stroke-linejoin="round"/>
            <path d="M3 8l9 5 9-5M12 13v8" stroke="currentColor" stroke-linejoin="round"/>
          </svg>
          <span>{{ t('nav.inventory') }}</span>
        </router-link>

        <router-link
          to="/orders"
          class="nav-item"
          :title="t('nav.orders')"
          @click="closeMobile"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M9 3h6a1 1 0 011 1v1h1a1 1 0 011 1v14a1 1 0 01-1 1H6a1 1 0 01-1-1V6a1 1 0 011-1h1V4a1 1 0 011-1z" stroke="currentColor" stroke-linejoin="round"/>
            <path d="M9 11l2 2 4-4" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span>{{ t('nav.orders') }}</span>
        </router-link>

        <router-link
          to="/spending"
          class="nav-item"
          :title="t('nav.finance')"
          @click="closeMobile"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <circle cx="12" cy="12" r="9" stroke="currentColor"/>
            <path d="M14.5 9a2.5 2.5 0 00-2.5-1.5c-1.4 0-2.5.9-2.5 2s1.1 1.6 2.5 2 2.5.9 2.5 2-1.1 2-2.5 2A2.5 2.5 0 019.5 16M12 6v1.5M12 16.5V18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span>{{ t('nav.finance') }}</span>
        </router-link>
      </nav>

      <nav class="nav-group">
        <div class="nav-label">Insights</div>

        <router-link
          to="/demand"
          class="nav-item"
          :title="t('nav.demandForecast')"
          @click="closeMobile"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M4 15l5-5 4 4 7-7" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M15 7h5v5" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span>{{ t('nav.demandForecast') }}</span>
        </router-link>

        <router-link
          to="/reports"
          class="nav-item"
          title="Reports"
          @click="closeMobile"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M5 21V10M12 21V4M19 21v-7" stroke="currentColor" stroke-linecap="round"/>
            <path d="M3 21h18" stroke="currentColor" stroke-linecap="round"/>
          </svg>
          <span>Reports</span>
        </router-link>
      </nav>

      <div class="sidebar__foot">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="app-main">
      <header class="page-header">
        <button
          class="sidebar__mobile-toggle"
          type="button"
          @click="toggleSidebar"
          aria-label="Open navigation"
        >
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M4 6h16M4 12h16M4 18h16" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
          </svg>
        </button>
        <FilterBar />
      </header>

      <main class="content">
        <router-view />
      </main>
    </div>

    <div
      v-if="mobileOpen && isMobile"
      class="sidebar-backdrop"
      @click="closeMobile"
    ></div>

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
import { ref, onMounted, onUnmounted, computed } from 'vue'
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

    // ---- Sidebar shell state (presentational only) ------------------------
    const collapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')
    const isMobile = ref(false)
    const mobileOpen = ref(false)

    let mqCollapse = null
    let mqMobile = null

    const syncViewport = () => {
      isMobile.value = mqMobile ? mqMobile.matches : false
      if (isMobile.value) {
        // Off-canvas drawer on small screens — start closed.
        mobileOpen.value = false
      } else if (mqCollapse && mqCollapse.matches) {
        // Auto-collapse to the icon rail on narrow viewports (<= 1024px).
        collapsed.value = true
      } else {
        // Wide screens: restore the persisted user preference.
        collapsed.value = localStorage.getItem('sidebar-collapsed') === 'true'
      }
    }

    const toggleSidebar = () => {
      if (isMobile.value) {
        mobileOpen.value = !mobileOpen.value
      } else {
        collapsed.value = !collapsed.value
        localStorage.setItem('sidebar-collapsed', String(collapsed.value))
      }
    }

    const closeMobile = () => {
      mobileOpen.value = false
    }

    onMounted(() => {
      mqCollapse = window.matchMedia('(max-width: 1024px)')
      mqMobile = window.matchMedia('(max-width: 640px)')
      mqCollapse.addEventListener('change', syncViewport)
      mqMobile.addEventListener('change', syncViewport)
      syncViewport()
    })

    onUnmounted(() => {
      if (mqCollapse) mqCollapse.removeEventListener('change', syncViewport)
      if (mqMobile) mqMobile.removeEventListener('change', syncViewport)
    })

    onMounted(loadTasks)

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      collapsed,
      isMobile,
      mobileOpen,
      toggleSidebar,
      closeMobile
    }
  }
}
</script>

<style>
/* ============================================================================
   SHELL CHROME  (sidebar + top bar)
   Layout primitives (.app-shell, .sidebar, .app-main, .page-header, .content,
   .nav-item, .filter-bar) come from the global design system. The rules below
   only add the collapse toggle, mobile off-canvas behaviour, and dock the
   filter bar into the sticky page header.
   ============================================================================ */

/* Collapse / expand control, styled as a muted nav row. */
.sidebar__toggle {
  width: 100%;
  border: 0;
  background: none;
  color: var(--nav-text-dim);
  font-family: var(--font-sans);
  font-size: var(--text-sm);
  cursor: pointer;
}
.sidebar__toggle:hover {
  background: var(--nav-hover);
  color: var(--nav-text-strong);
}
.sidebar__toggle-icon {
  transition: transform var(--transition-base);
}
.app-shell.is-collapsed .sidebar__toggle-icon {
  transform: rotate(180deg);
}

/* Dock the global filter bar inside the sticky page header. */
.app-main > .page-header {
  margin-bottom: 0;
}
.page-header .filter-bar {
  flex: 1;
  padding: 0;
  background: transparent;
  border-bottom: 0;
}

/* Footer stack for language + profile controls. */
.sidebar__foot {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

/* Collapsed rail: hide footer labels/chevrons, center the controls. */
.sidebar.is-collapsed .sidebar__foot .language-label,
.sidebar.is-collapsed .sidebar__foot .profile-name,
.sidebar.is-collapsed .sidebar__foot .language-button .chevron,
.sidebar.is-collapsed .sidebar__foot .profile-button .chevron {
  display: none;
}
.sidebar.is-collapsed .sidebar__foot .language-button,
.sidebar.is-collapsed .sidebar__foot .profile-button {
  justify-content: center;
  padding-inline: var(--space-2);
}
.sidebar.is-collapsed .sidebar__foot .dropdown-menu {
  left: 0;
}

/* Mobile-only hamburger that opens the off-canvas drawer. */
.sidebar__mobile-toggle {
  display: none;
  align-items: center;
  justify-content: center;
  width: 38px;
  height: 38px;
  flex: none;
  border: 1px solid var(--border-strong);
  border-radius: var(--radius-md);
  background: var(--surface);
  color: var(--ink-2);
  box-shadow: var(--shadow-1);
  cursor: pointer;
}
.sidebar__mobile-toggle svg {
  width: 20px;
  height: 20px;
}

/* Dimmed scrim behind the mobile drawer. */
.sidebar-backdrop {
  position: fixed;
  inset: 0;
  z-index: var(--z-dropdown);
  background: var(--ink);
  opacity: 0.5;
  border: 0;
}

/* Off-canvas drawer behaviour on small screens. */
@media (max-width: 640px) {
  .app-shell,
  .app-shell.is-collapsed {
    grid-template-columns: 1fr;
  }
  .sidebar__mobile-toggle {
    display: inline-flex;
  }
  .sidebar {
    position: fixed;
    inset: 0 auto 0 0;
    width: var(--sidebar-w);
    z-index: var(--z-overlay);
    transform: translateX(-100%);
    transition: transform var(--transition-base);
  }
  .app-shell.is-mobile-open .sidebar {
    transform: translateX(0);
    box-shadow: var(--shadow-3);
  }
}

/* Legacy view helpers were removed. The design system is the single source of
   truth for the redesigned views; view/modal-specific rules that were still in
   use have been relocated into the scoped styles of the components that use
   them (Backlog.vue and the detail modals). */
</style>
