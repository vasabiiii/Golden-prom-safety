<script setup>
import { computed, ref } from 'vue'
import About from './components/About.vue'
import Auth from './components/Auth.vue'
import Contacts from './components/Contacts.vue'
import Courses from './components/Courses.vue'
import Header from './components/Header.vue'

const activePage = ref('about')
const currentUser = ref(null)

const currentPageComponent = computed(() => {
  if (activePage.value === 'courses') return Courses
  if (activePage.value === 'contacts') return Contacts
  if (activePage.value === 'auth') return Auth
  return About
})

const onNavigate = (page) => {
  if (page === 'profile' && !currentUser.value) {
    activePage.value = 'auth'
    return
  }

  activePage.value = page
}

const onLogin = (user) => {
  currentUser.value = user
  activePage.value = 'profile'
}

const onLogout = () => {
  currentUser.value = null
  activePage.value = 'about'
}
</script>

<template>
  <div class="app-shell">
    <Header :current-page="activePage" :current-user="currentUser" @navigate="onNavigate" @logout="onLogout" />

    <main class="content">
      <section v-if="activePage === 'profile'" class="profile-card">
        <h1>Личный кабинет</h1>
        <p>
          Добро пожаловать, <strong>{{ currentUser?.name }}</strong
          >! Здесь будет ваш персональный прогресс и настройки.
        </p>
        <div class="profile-meta">
          <span><b>Email:</b> {{ currentUser?.email }}</span>
          <span><b>Роль:</b> {{ currentUser?.role }}</span>
        </div>
        <button
          v-if="currentUser?.role === 'admin'"
          type="button"
          class="admin-btn"
          @click="onNavigate('auth')"
        >
          Управление пользователями
        </button>
      </section>

      <component
        :is="currentPageComponent"
        v-else
        :current-user="currentUser"
        @login="onLogin"
      />
    </main>
  </div>
</template>

<style scoped>
.app-shell {
  min-height: 100vh;
  background: linear-gradient(180deg, #f8fafc 0%, #eef2ff 100%);
  color: #0f172a;
  font-family: Inter, 'Segoe UI', Roboto, Arial, sans-serif;
}

.content {
  width: 100%;
  max-width: 1100px;
  box-sizing: border-box;
  margin: 0 auto;
  padding: 32px 22px 40px;
}

.profile-card {
  width: 100%;
  max-width: 100%;
  box-sizing: border-box;
  background: #ffffff;
  border-radius: 18px;
  padding: 30px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 16px 40px rgba(15, 23, 42, 0.08);
}

.profile-card h1 {
  margin: 0 0 12px;
  font-size: 2rem;
}

.profile-card p {
  margin: 0 0 16px;
  color: #475569;
  line-height: 1.7;
}

.profile-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.profile-meta span {
  background: #f1f5f9;
  color: #334155;
  border-radius: 999px;
  padding: 7px 12px;
  font-size: 0.9rem;
}

.admin-btn {
  margin-top: 18px;
  border: none;
  border-radius: 10px;
  padding: 10px 14px;
  background: #0f172a;
  color: #ffffff;
  cursor: pointer;
}

@media (max-width: 768px) {
  .content {
    padding: 18px 12px 28px;
  }

  .profile-card {
    padding: 22px 16px;
    border-radius: 14px;
  }

  .profile-card h1 {
    font-size: 1.55rem;
  }
}
</style>
