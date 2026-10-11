<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  currentPage: {
    type: String,
    required: true,
  },
  currentUser: {
    type: Object,
    default: null,
  },
})

const emit = defineEmits(['navigate', 'logout'])

const navItems = [
  { key: 'about', label: 'О нас' },
  { key: 'courses', label: 'Обучение' },
  { key: 'contacts', label: 'Контакты' },
]

const isMenuOpen = ref(false)
const authLabel = computed(() => (props.currentUser ? `Кабинет (${props.currentUser.name})` : 'Войти'))

const onNavigate = (page) => {
  isMenuOpen.value = false
  emit('navigate', page)
}

const onLogout = () => {
  isMenuOpen.value = false
  emit('logout')
}

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value
}

const onAuthClick = () => {
  onNavigate(props.currentUser ? 'profile' : 'auth')
}
</script>

<template>
  <div class="header-shell">
    <header class="header">
      <button class="brand" type="button" @click="onNavigate('about')">EduSpace</button>

      <nav class="nav-desktop">
        <button
          v-for="item in navItems"
          :key="item.key"
          type="button"
          class="nav-link"
          :class="{ active: currentPage === item.key }"
          @click="onNavigate(item.key)"
        >
          {{ item.label }}
        </button>
      </nav>

      <div class="controls">
        <button type="button" class="btn primary auth-btn" @click="onAuthClick">
          <span class="auth-label-full">{{ authLabel }}</span>
          <span class="auth-label-mobile">{{ currentUser ? 'Кабинет' : 'Вход' }}</span>
        </button>

        <button
          type="button"
          class="menu-toggle"
          :aria-label="isMenuOpen ? 'Закрыть меню' : 'Открыть меню'"
          @click="toggleMenu"
        >
          <span class="menu-icon">{{ isMenuOpen ? '×' : '☰' }}</span>
        </button>
      </div>
    </header>

    <nav class="nav-mobile" :class="{ open: isMenuOpen }">
      <button
        v-for="item in navItems"
        :key="item.key"
        type="button"
        class="mobile-link"
        :class="{ active: currentPage === item.key }"
        @click="onNavigate(item.key)"
      >
        {{ item.label }}
      </button>

      <button v-if="currentUser" type="button" class="mobile-link logout-link" @click="onLogout">Выйти</button>
    </nav>
  </div>
</template>

<style scoped>
.header-shell {
  position: sticky;
  top: 0;
  z-index: 100;
  background-color: #ffffff;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  width: 100%;
  box-sizing: border-box;
  padding: 14px 22px;
  border-bottom: 1px solid #e2e8f0;
  background-color: #ffffff;
  position: relative;
  z-index: 100;
}

.brand {
  border: none;
  background: transparent;
  font-size: 1.22rem;
  font-weight: 800;
  color: #1e293b;
  cursor: pointer;
  line-height: 1;
}

.nav-desktop {
  display: flex;
  align-items: center;
  justify-content: center;
  flex: 1;
  gap: 8px;
}

.nav-link {
  border: none;
  background: transparent;
  border-radius: 999px;
  padding: 8px 14px;
  font-size: 0.95rem;
  color: #475569;
  cursor: pointer;
  transition: all 0.2s ease;
}

.nav-link:hover,
.nav-link.active {
  background: #f1f5f9;
  color: #0f172a;
}

.controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn {
  border: none;
  border-radius: 10px;
  padding: 9px 14px;
  font-size: 0.9rem;
  cursor: pointer;
}

.primary {
  background: #2563eb;
  color: #ffffff;
  box-shadow: 0 8px 20px rgba(37, 99, 235, 0.2);
}

.auth-label-mobile {
  display: none;
}

.menu-toggle {
  display: none;
  width: 38px;
  height: 38px;
  border-radius: 10px;
  border: 1px solid #cbd5e1;
  background: #f8fafc;
  color: #0f172a;
  cursor: pointer;
}

.menu-icon {
  font-size: 1.2rem;
  line-height: 1;
}

.nav-mobile {
  display: none;
}

@media (max-width: 767px) {
  .header {
    padding: 12px;
  }

  .nav-desktop {
    display: none;
  }

  .menu-toggle {
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }

  .auth-btn {
    padding: 8px 10px;
    font-size: 0.82rem;
    white-space: nowrap;
  }

  .auth-label-full {
    display: none;
  }

  .auth-label-mobile {
    display: inline;
  }

  .nav-mobile {
    position: absolute;
    top: 100%;
    left: 0;
    width: 100%;
    box-sizing: border-box;
    z-index: 99;
    display: flex;
    flex-direction: column;
    gap: 8px;
    padding: 10px 12px 14px;
    background: #ffffff !important;
    border-bottom: 1px solid #e2e8f0;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
    transform: translateY(-120%);
    opacity: 0;
    pointer-events: none;
    transition: transform 0.22s ease, opacity 0.22s ease;
  }

  .nav-mobile.open {
    transform: translateY(0);
    opacity: 1;
    pointer-events: auto;
  }

  .mobile-link {
    border: 1px solid #e2e8f0;
    background: #ffffff;
    color: #0f172a;
    border-radius: 12px;
    min-height: 46px;
    text-align: left;
    padding: 0 14px;
    font-size: 1rem;
    cursor: pointer;
  }

  .mobile-link.active {
    background: #eff6ff;
    border-color: #bfdbfe;
    color: #1d4ed8;
  }

  .logout-link {
    background: #fef2f2;
    border-color: #fecaca;
    color: #b91c1c;
  }
}
</style>
