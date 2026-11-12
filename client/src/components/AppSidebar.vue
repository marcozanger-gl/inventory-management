<template>
  <aside
    ref="rootEl"
    class="sidebar"
    :class="{ 'is-collapsed': collapsed, 'is-open': open }"
  >
    <div class="sidebar-header">
      <div class="brand">
        <span class="brand-mark" aria-hidden="true">{{ brandInitial }}</span>
        <div class="brand-text">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>
      <button
        type="button"
        class="collapse-btn"
        :aria-expanded="String(!collapsed)"
        :aria-label="collapsed ? t('nav.expand') : t('nav.collapse')"
        @click="$emit('toggle-collapse')"
      >
        <svg
          class="collapse-icon"
          :class="{ flipped: collapsed }"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
        >
          <path d="M15 6l-6 6 6 6" />
        </svg>
      </button>
    </div>

    <nav class="sidebar-nav" :aria-label="t('nav.primary')">
      <router-link
        v-for="item in items"
        :key="item.path"
        :to="item.path"
        class="nav-link"
        @click="$emit('close')"
      >
        <svg
          class="nav-icon"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
          v-html="icons[item.icon]"
        ></svg>
        <span class="nav-label">{{ t(item.labelKey) }}</span>
        <span class="nav-tooltip" aria-hidden="true">{{ t(item.labelKey) }}</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher :compact="isCompact" :placement="placement" />
      <ProfileMenu
        :compact="isCompact"
        :placement="placement"
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </aside>
</template>

<script>
import { ref, computed, watch, nextTick, onMounted, onBeforeUnmount } from 'vue'
import { useI18n } from '../composables/useI18n'
import LanguageSwitcher from './LanguageSwitcher.vue'
import ProfileMenu from './ProfileMenu.vue'

// Static, trusted SVG inner markup (viewBox 0 0 24 24, stroked)
const icons = {
  layout: '<rect x="3" y="3" width="7" height="9"/><rect x="14" y="3" width="7" height="5"/><rect x="14" y="12" width="7" height="9"/><rect x="3" y="16" width="7" height="5"/>',
  package: '<path d="M21 8l-9-5-9 5v8l9 5 9-5V8z"/><path d="M3 8l9 5 9-5"/><path d="M12 13v8"/>',
  clipboard: '<rect x="8" y="2" width="8" height="4" rx="1"/><path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"/><path d="M9 12h6M9 16h6"/>',
  coins: '<circle cx="12" cy="12" r="9"/><path d="M12 6v12"/><path d="M15 9c-.6-.8-1.7-1.2-3-1.2-1.6 0-2.8.7-2.8 1.9 0 2.6 5.8 1.3 5.8 3.9 0 1.2-1.2 1.9-2.9 1.9-1.3 0-2.4-.4-3.1-1.3"/>',
  trend: '<path d="M3 17l6-6 4 4 8-8"/><path d="M15 7h6v6"/>',
  file: '<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><path d="M14 2v6h6"/><path d="M8 13h8M8 17h5"/>'
}

const items = [
  { path: '/', labelKey: 'nav.overview', icon: 'layout' },
  { path: '/inventory', labelKey: 'nav.inventory', icon: 'package' },
  { path: '/orders', labelKey: 'nav.orders', icon: 'clipboard' },
  { path: '/spending', labelKey: 'nav.finance', icon: 'coins' },
  { path: '/demand', labelKey: 'nav.demandForecast', icon: 'trend' },
  { path: '/reports', labelKey: 'nav.reports', icon: 'file' }
]

export default {
  name: 'AppSidebar',
  components: { LanguageSwitcher, ProfileMenu },
  props: {
    collapsed: { type: Boolean, default: false },
    open: { type: Boolean, default: false }
  },
  emits: ['toggle-collapse', 'close', 'show-profile-details', 'show-tasks'],
  setup(props) {
    const { t } = useI18n()
    const rootEl = ref(null)

    // Collapsed only applies on desktop; the drawer always shows labels
    // Seeded synchronously so the first paint matches the viewport (no flash)
    const DESKTOP_QUERY = '(min-width: 768px)'
    const isDesktop = ref(
      typeof window !== 'undefined' && window.matchMedia
        ? window.matchMedia(DESKTOP_QUERY).matches
        : true
    )
    let mql = null
    const onMq = (e) => { isDesktop.value = e.matches }

    onMounted(() => {
      if (typeof window !== 'undefined' && window.matchMedia) {
        mql = window.matchMedia(DESKTOP_QUERY)
        isDesktop.value = mql.matches
        mql.addEventListener('change', onMq)
      }
    })
    onBeforeUnmount(() => {
      if (mql) mql.removeEventListener('change', onMq)
    })

    const isCompact = computed(() => props.collapsed && isDesktop.value)
    // Expanded: open upward from the footer; collapsed: open to the right, bottom aligned
    const placement = computed(() => (isCompact.value ? 'right-end' : 'top-start'))
    const brandInitial = computed(() => (t('nav.companyName') || '').trim().charAt(0))

    // Move focus to the first link when the drawer opens
    watch(() => props.open, async (isOpen) => {
      if (!isOpen) return
      await nextTick()
      const first = rootEl.value && rootEl.value.querySelector('.nav-link')
      if (first) first.focus()
    })

    return { t, rootEl, icons, items, isCompact, placement, brandInitial }
  }
}
</script>

<style scoped>
.sidebar {
  display: flex;
  flex-direction: column;
  background: var(--sidebar-bg);
  color: var(--sidebar-text);
  border-right: 1px solid var(--sidebar-border);
  font-family: var(--font-sans);
  z-index: var(--z-sidebar);
  /* Mobile default: off-canvas drawer */
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  width: min(280px, 85vw);
  transform: translateX(-100%);
  /* Delay the visibility flip so the slide-out stays visible */
  transition: transform 0.2s ease, visibility 0s linear 0.2s;
  overflow-y: auto;
  visibility: hidden;
}

.sidebar.is-open {
  transform: translateX(0);
  visibility: visible;
  transition: transform 0.2s ease, visibility 0s;
  /* Must sit above the overlay (--z-overlay) or taps hit the overlay */
  z-index: calc(var(--z-overlay) + 1);
}

/* Drawer only: dropdowns must fit the drawer width */
@media (max-width: 767.98px) {
  .sidebar-footer :deep(.dropdown-menu) {
    min-width: 0;
    left: 0;
    right: 0;
    max-width: none;
  }
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-2);
  padding: var(--space-4);
  border-bottom: 1px solid var(--sidebar-border);
  min-height: var(--topbar-height);
}

.brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  min-width: 0;
}

.brand-mark {
  display: none;
  flex-shrink: 0;
  width: 32px;
  height: 32px;
  align-items: center;
  justify-content: center;
  border-radius: var(--radius-sm);
  background: var(--sidebar-active-bg);
  color: var(--sidebar-active-text);
  font-weight: var(--weight-bold);
  font-size: var(--text-sm);
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  color: var(--sidebar-active-text);
  font-size: var(--text-base);
  font-weight: var(--weight-semibold);
  letter-spacing: var(--tracking-tight);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  color: var(--sidebar-text-muted);
  font-size: var(--text-xs);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.collapse-btn {
  display: none;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: none;
  border: none;
  border-radius: var(--radius-sm);
  color: var(--sidebar-text-muted);
  cursor: pointer;
}

.collapse-btn:hover {
  background: var(--sidebar-hover-bg);
  color: var(--sidebar-active-text);
}

.collapse-icon {
  transition: transform 0.2s ease;
}

.collapse-icon.flipped {
  transform: rotate(180deg);
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-3) var(--space-2);
  flex: 1;
}

.nav-link {
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  min-height: 40px;
  border-radius: var(--radius-md);
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
}

.nav-link:hover {
  background: var(--sidebar-hover-bg);
}

.nav-link.router-link-exact-active {
  background: var(--sidebar-active-bg);
  color: var(--sidebar-active-text);
}

.nav-icon {
  flex-shrink: 0;
}

.nav-label {
  white-space: nowrap;
}

.nav-tooltip {
  display: none;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3) var(--space-2);
  border-top: 1px solid var(--sidebar-border);
}

/* Footer triggers sit on a dark surface */
.sidebar-footer :deep(.profile-button),
.sidebar-footer :deep(.language-button) {
  width: 100%;
  background: transparent;
  border-color: var(--sidebar-border);
  color: var(--sidebar-text);
}

.sidebar-footer :deep(.profile-button:hover),
.sidebar-footer :deep(.language-button:hover) {
  background: var(--sidebar-hover-bg);
  border-color: var(--sidebar-hover-bg);
}

.sidebar-footer :deep(.profile-name) {
  color: var(--sidebar-text);
}

.sidebar-footer :deep(.globe-icon),
.sidebar-footer :deep(.chevron) {
  color: var(--sidebar-text-muted);
}

.sidebar-footer :deep(.chevron) {
  margin-left: auto;
}

/* Visible keyboard focus on every control */
.nav-link:focus-visible,
.collapse-btn:focus-visible,
.sidebar-footer :deep(button:focus-visible) {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

/* Desktop: sticky column, expanded or collapsed */
@media (min-width: 768px) {
  .sidebar,
  .sidebar.is-open {
    z-index: var(--z-sidebar);
  }

  .sidebar {
    position: sticky;
    top: 0;
    height: 100vh;
    flex-shrink: 0;
    width: var(--sidebar-width);
    transform: none;
    visibility: visible;
    overflow: visible; /* lets tooltips and dropdowns escape */
    transition: width 0.2s ease;
  }

  .collapse-btn {
    display: inline-flex;
  }

  .sidebar.is-collapsed {
    width: var(--sidebar-width-collapsed);
  }

  .sidebar.is-collapsed .sidebar-header {
    flex-direction: column;
    justify-content: center;
    padding: var(--space-3) var(--space-2);
  }

  .sidebar.is-collapsed .brand-mark {
    display: inline-flex;
  }

  /* Hidden visually, still read by screen readers */
  .sidebar.is-collapsed .brand-text,
  .sidebar.is-collapsed .nav-label {
    position: absolute;
    width: 1px;
    height: 1px;
    margin: -1px;
    padding: 0;
    overflow: hidden;
    clip: rect(0 0 0 0);
    white-space: nowrap;
    border: 0;
  }

  .sidebar.is-collapsed .nav-link {
    justify-content: center;
    padding-left: 0;
    padding-right: 0;
  }

  .sidebar.is-collapsed .nav-tooltip {
    display: block;
    position: absolute;
    left: calc(100% + var(--space-2));
    top: 50%;
    transform: translateY(-50%);
    padding: var(--space-1) var(--space-2);
    background: var(--sidebar-bg);
    border: 1px solid var(--sidebar-border);
    border-radius: var(--radius-sm);
    color: var(--sidebar-active-text);
    font-size: var(--text-xs);
    white-space: nowrap;
    box-shadow: var(--shadow-md);
    pointer-events: none;
    opacity: 0;
    visibility: hidden;
    z-index: var(--z-dropdown);
  }

  .sidebar.is-collapsed .nav-link:hover .nav-tooltip,
  .sidebar.is-collapsed .nav-link:focus-visible .nav-tooltip {
    opacity: 1;
    visibility: visible;
  }

  .sidebar.is-collapsed .sidebar-footer :deep(.profile-button),
  .sidebar.is-collapsed .sidebar-footer :deep(.language-button) {
    justify-content: center;
  }

  .sidebar.is-collapsed .sidebar-footer {
    align-items: stretch;
  }
}

@media (prefers-reduced-motion: reduce) {
  .sidebar,
  .sidebar.is-open,
  .collapse-icon,
  .nav-link {
    transition: none;
  }
}
</style>
