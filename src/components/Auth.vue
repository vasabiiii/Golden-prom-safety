<script setup>
import { reactive, ref } from 'vue'

const emit = defineEmits(['login'])

const users = ref([
  { id: 1, name: 'Тестовый пользователь', email: 'test@test.test', password: '12345678', role: 'user' },
  { id: 2, name: 'Анна', email: 'admin@example.com', password: 'admin123', role: 'admin' },
])

const activeTab = ref('login')
const registerMethod = ref('phone')
const message = ref('')
const messageType = ref('info')
const errors = reactive({
  firstName: '',
  lastName: '',
  phone: '',
  email: '',
  password: '',
  confirmPassword: '',
})

const loginForm = reactive({
  email: '',
  password: '',
})

const registerForm = reactive({
  firstName: '',
  lastName: '',
  email: '',
  password: '',
  confirmPassword: '',
})

const formattedPhone = ref('+7 ')
const rawPhoneValue = ref('') // сюда будет сохраняться чистый номер без скобок для бэкенда Go

// Переменная флаг, чтобы понимать, что мы стираем
let isBackspace = false

const clearErrors = () => {
  errors.firstName = ''
  errors.lastName = ''
  errors.phone = ''
  errors.email = ''
  errors.password = ''
  errors.confirmPassword = ''
}

const clearFieldError = (field) => {
  errors[field] = ''
}

const setRegisterMethod = (method) => {
  registerMethod.value = method
  if (method === 'phone') {
    clearFieldError('email')
  } else {
    clearFieldError('phone')
  }
}

const handleKeyDown = (event) => {
  if (event.key === 'Backspace') {
    isBackspace = true
    let input = event.target.value

    // Если на конце форматирующий символ, принудительно удаляем его и цифру перед ним
    if (input.endsWith('-') || input.endsWith(' ') || input.endsWith(')')) {
      event.preventDefault() // останавливаем стандартное забагованное стирание

      // Забираем чистые цифры, удаляем последнюю и принудительно пересобираем строку
      let digits = input.substring(2).replace(/\D/g, '')
      digits = digits.substring(0, digits.length - 1)

      updateMask(digits)
    }
  } else {
    isBackspace = false
  }
}

const handlePhoneInput = (event) => {
  if (isBackspace) return // если стираем через keydown, обычный инпут не трогаем

  let input = event.target.value
  if (!input.startsWith('+7')) {
    formattedPhone.value = '+7 '
    clearFieldError('phone')
    return
  }

  let digits = input.substring(2).replace(/\D/g, '')
  if (digits.length > 10) digits = digits.substring(0, 10)

  clearFieldError('phone')
  updateMask(digits)
}

// Вынесли сборку маски в отдельную чистую функцию
const updateMask = (digits) => {
  rawPhoneValue.value = '+7' + digits

  let result = '+7 '
  if (digits.length > 0) result += '(' + digits.substring(0, 3)
  if (digits.length >= 3) result += ') ' + digits.substring(3, 6)
  if (digits.length >= 6) result += '-' + digits.substring(6, 8)
  if (digits.length >= 8) result += '-' + digits.substring(8, 10)

  formattedPhone.value = result
}

const setMessage = (text, type = 'info') => {
  message.value = text
  messageType.value = type
}

const clearMessage = () => {
  message.value = ''
}

const onLogin = () => {
  clearMessage()

  const foundUser = users.value.find(
    (user) => user.email === loginForm.email.trim() && user.password === loginForm.password,
  )

  if (!foundUser) {
    setMessage('Неверный email или пароль.', 'error')
    return
  }

  emit('login', { ...foundUser })
  setMessage(`Вы вошли как ${foundUser.name}.`, 'success')
}

const onRegister = () => {
  clearMessage()
  clearErrors()

  const firstName = registerForm.firstName.trim()
  const lastName = registerForm.lastName.trim()
  const normalizedEmail = registerForm.email.trim()
  const normalizedPassword = registerForm.password.trim()
  const normalizedConfirmPassword = registerForm.confirmPassword.trim()

  if (!firstName) {
    errors.firstName = 'Поле должно быть заполнено'
  }

  if (!lastName) {
    errors.lastName = 'Поле должно быть заполнено'
  }

  if (registerMethod.value === 'phone') {
    if (!rawPhoneValue.value || rawPhoneValue.value === '+7' || !/^\+7\d{10}$/.test(rawPhoneValue.value)) {
      errors.phone = 'Поле должно быть заполнено'
    }
  } else if (!normalizedEmail) {
    errors.email = 'Поле должно быть заполнено'
  }

  if (!normalizedPassword) {
    errors.password = 'Поле должно быть заполнено'
  } else if (normalizedPassword.length < 6) {
    errors.password = 'Минимум 6 символов'
  }

  if (normalizedConfirmPassword !== normalizedPassword) {
    errors.confirmPassword = 'Пароли не совпадают'
  }

  const hasErrors = Object.values(errors).some((value) => value)
  if (hasErrors) {
    return
  }

  if (registerMethod.value === 'phone') {
    const phoneExists = users.value.some((user) => user.phone === rawPhoneValue.value)
    if (phoneExists) {
      errors.phone = 'Пользователь с таким номером уже существует'
      return
    }
  } else {
    const exists = users.value.some((user) => user.email === normalizedEmail)
    if (exists) {
      errors.email = 'Пользователь с таким email уже существует'
      return
    }
  }

  const newUser = {
    id: Date.now(),
    name: `${firstName} ${lastName}`,
    phone: registerMethod.value === 'phone' ? rawPhoneValue.value : '',
    email: registerMethod.value === 'email' ? normalizedEmail : '',
    password: normalizedPassword,
    role: 'user',
  }

  users.value.push(newUser)
  registerForm.firstName = ''
  registerForm.lastName = ''
  formattedPhone.value = '+7 '
  rawPhoneValue.value = ''
  registerForm.email = ''
  registerForm.password = ''
  registerForm.confirmPassword = ''
  clearErrors()
  setMessage('Новый пользователь успешно зарегистрирован.', 'success')
}
</script>

<template>
  <section class="auth-card">
    <h1>Авторизация</h1>

    <div class="tabs">
      <span class="tab-indicator" :class="{ shifted: activeTab === 'register' }"></span>
      <button
        type="button"
        class="tab"
        :class="{ active: activeTab === 'login' }"
        @click="activeTab = 'login'"
      >
        Вход
      </button>
      <button
        type="button"
        class="tab"
        :class="{ active: activeTab === 'register' }"
        @click="activeTab = 'register'"
      >
        Регистрация
      </button>
    </div>

    <form v-if="activeTab === 'login'" class="form" @submit.prevent="onLogin" novalidate>
      <label>
        <span class="field-label">Email</span>
        <input v-model="loginForm.email" type="email" placeholder="test@test.test" required />
      </label>
      <label>
        <span class="field-label">Пароль</span>
        <input v-model="loginForm.password" type="password" placeholder="Введите пароль" required />
      </label>
      <button type="submit" class="submit-btn">Войти</button>
    </form>

    <form v-else class="form" @submit.prevent="onRegister" novalidate>
      <div class="name-row">
        <label>
          <span class="field-label">Имя</span>
          <input
            v-model="registerForm.firstName"
            :class="{ 'input-error': errors.firstName }"
            type="text"
            placeholder="Имя"
            @input="clearFieldError('firstName')"
            required
          />
          <span v-if="errors.firstName" class="error-text">{{ errors.firstName }}</span>
        </label>
        <label>
          <span class="field-label">Фамилия</span>
          <input
            v-model="registerForm.lastName"
            :class="{ 'input-error': errors.lastName }"
            type="text"
            placeholder="Фамилия"
            @input="clearFieldError('lastName')"
            required
          />
          <span v-if="errors.lastName" class="error-text">{{ errors.lastName }}</span>
        </label>
      </div>

      <div class="register-method-tabs">
        <span class="method-indicator" :class="{ shifted: registerMethod === 'email' }"></span>
        <button
          type="button"
          class="method-btn"
          :class="{ active: registerMethod === 'phone' }"
          @click="setRegisterMethod('phone')"
        >
          По номеру
        </button>
        <button
          type="button"
          class="method-btn"
          :class="{ active: registerMethod === 'email' }"
          @click="setRegisterMethod('email')"
        >
          По Email
        </button>
      </div>

      <label v-if="registerMethod === 'phone'">
        <span class="field-label">Номер телефона</span>
        <input
          type="tel"
          v-model="formattedPhone"
          :class="{ 'input-error': errors.phone }"
          @input="handlePhoneInput"
          @keydown="handleKeyDown"
          placeholder="+7 (705) 123-45-67"
          inputmode="tel"
          required
        />
        <span v-if="errors.phone" class="error-text">{{ errors.phone }}</span>
      </label>

      <label v-else>
        <span class="field-label">Email</span>
        <input
          v-model="registerForm.email"
          :class="{ 'input-error': errors.email }"
          type="email"
          placeholder="new@example.com"
          @input="clearFieldError('email')"
          required
        />
        <span v-if="errors.email" class="error-text">{{ errors.email }}</span>
      </label>
      <label>
        <span class="field-label">Пароль</span>
        <input
          v-model="registerForm.password"
          :class="{ 'input-error': errors.password }"
          type="password"
          placeholder="Минимум 6 символов"
          minlength="6"
          @input="clearFieldError('password')"
          required
        />
        <span v-if="errors.password" class="error-text">{{ errors.password }}</span>
      </label>
      <label>
        <span class="field-label">Повторите пароль</span>
        <input
          v-model="registerForm.confirmPassword"
          :class="{ 'input-error': errors.confirmPassword }"
          type="password"
          placeholder="Повторите пароль"
          minlength="6"
          @input="clearFieldError('confirmPassword')"
          required
        />
        <span v-if="errors.confirmPassword" class="error-text">{{ errors.confirmPassword }}</span>
      </label>
      <button type="submit" class="submit-btn">Зарегистрироваться</button>
    </form>

    <p v-if="message" class="message" :class="messageType">{{ message }}</p>
  </section>
</template>

<style scoped>
.auth-card {
  width: 100%;
  max-width: 400px;
  box-sizing: border-box;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(12px);
  border: 1px solid #e2e8f0;
  box-shadow: 0 20px 40px rgba(13, 37, 63, 0.08);
  border-radius: 18px;
  padding: 30px;
  overflow: hidden;
}

h1 {
  margin: 0 0 18px;
  color: #0f172a;
  font-size: 1.7rem;
}

.tabs {
  position: relative;
  display: grid;
  grid-template-columns: 1fr 1fr;
  padding: 4px;
  border-radius: 12px;
  background-color: #f1f5f9 !important;
  border: 1px solid #e2e8f0;
  margin-bottom: 18px;
  overflow: hidden;
}

.tab-indicator {
  position: absolute;
  top: 4px;
  left: 4px;
  width: calc(50% - 4px);
  height: calc(100% - 8px);
  border-radius: 10px;
  background: linear-gradient(135deg, #0d253f 0%, #1f4c7f 72%, #c9a75e 100%);
  box-shadow: none;
  transition: transform 0.3s ease;
}

.tab-indicator.shifted {
  transform: translateX(100%);
}

.tab {
  position: relative;
  z-index: 1;
  border: none;
  border-radius: 10px;
  padding: 9px 14px;
  background: transparent !important;
  background-color: transparent !important;
  color: #64748b;
  cursor: pointer;
  font-weight: 600;
  transition: color 0.3s ease, background-color 0.3s ease;
  -webkit-tap-highlight-color: transparent;
}

.tab.active {
  background: transparent !important;
  background-color: transparent !important;
  color: #ffffff;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 14px;
  width: 100%;
  min-width: 0;
}

.name-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: 16px;
  width: 100%;
  min-width: 0;
}

.name-row > label {
  min-width: 0;
  width: 100%;
}

.register-method-tabs {
  position: relative;
  display: grid;
  grid-template-columns: 1fr 1fr;
  width: 100%;
  background-color: #f1f5f9 !important;
  border: none;
  border-radius: 8px;
  padding: 4px;
  overflow: hidden;
}

.method-indicator {
  position: absolute;
  top: 4px;
  left: 4px;
  width: calc(50% - 4px);
  height: calc(100% - 8px);
  border-radius: 6px;
  background: #ffffff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
  transform: translateX(0);
  transition:
    transform 0.3s cubic-bezier(0.4, 0, 0.2, 1),
    width 0.3s ease;
}

.method-indicator.shifted {
  transform: translateX(100%);
}

.method-btn {
  position: relative;
  z-index: 1;
  border: none;
  border-radius: 6px;
  width: 100%;
  min-height: 38px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  color: #64748b;
  font-size: 12px;
  font-weight: 600;
  padding: 8px 10px;
  cursor: pointer;
  text-decoration: none !important;
  transition: color 0.3s ease !important;
  outline: none !important;
  box-shadow: none !important;
  -webkit-tap-highlight-color: transparent;
}

.method-btn.active {
  color: #0f172a;
  font-weight: 700;
  text-decoration: none !important;
}

.method-btn:focus,
.method-btn:focus-visible,
.method-btn:active {
  outline: none !important;
  box-shadow: none !important;
}

label {
  display: flex;
  flex-direction: column;
  gap: 0;
  width: 100%;
  min-width: 0;
}

.field-label {
  text-transform: uppercase;
  font-size: 11px;
  font-weight: 700;
  color: #64748b;
  letter-spacing: 0.05em;
  margin-bottom: 4px;
}

input,
textarea {
  display: block;
  border: 2px solid #cbd5e1 !important;
  border-radius: 6px;
  padding: 8px 12px;
  font-size: 0.95rem;
  width: 100% !important;
  max-width: 100% !important;
  min-width: 0 !important;
  box-sizing: border-box !important;
  outline: none;
  transition: all 0.2s ease-in-out;
  background-color: #ffffff !important;
  color: #0f172a;
}

input::placeholder,
textarea::placeholder {
  color: #94a3b8;
}

input:focus,
textarea:focus {
  background-color: #ffffff;
  border-color: #475569 !important;
  outline: none !important;
  box-shadow: none !important;
}

input:focus-visible,
textarea:focus-visible {
  outline: none !important;
  box-shadow: none !important;
  border-color: #475569 !important;
}

.error-text {
  color: #ef4444;
  font-size: 11px;
  margin-top: 4px;
}

.input-error {
  border: 2px solid #ef4444 !important;
}

input.input-error:focus,
input.input-error:focus-visible,
textarea.input-error:focus,
textarea.input-error:focus-visible {
  border-color: #ef4444 !important;
  outline: none !important;
  box-shadow: none !important;
}

.submit-btn {
  margin-top: 6px;
  border: none;
  border-radius: 10px;
  padding: 11px 14px;
  background: linear-gradient(135deg, #0d253f 0%, #1f4c7f 100%);
  color: #ffffff;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 12px 24px rgba(13, 37, 63, 0.2);
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1), box-shadow 0.3s ease;
  -webkit-tap-highlight-color: transparent;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 18px 30px rgba(13, 37, 63, 0.3);
}

button:focus,
button:focus-visible,
.tab:focus,
.tab:focus-visible {
  outline: none !important;
  box-shadow: none !important;
}

button:active,
.tab:active,
.submit-btn:active {
  outline: none !important;
  box-shadow: none !important;
}

.message {
  margin: 16px 0 0;
  padding: 10px 12px;
  border-radius: 10px;
  font-size: 0.9rem;
}

.message.info {
  background: #eff6ff;
  color: #1d4ed8;
}

.message.success {
  background: #ecfdf3;
  color: #166534;
}

.message.error {
  background: #fef2f2;
  color: #b91c1c;
}

@media (max-width: 768px) {
  .auth-card {
    max-width: 100%;
    padding: 22px 16px;
    border-radius: 14px;
  }

  h1 {
    font-size: 1.45rem;
  }

  label {
    font-size: 0.95rem;
  }

  input,
  .submit-btn {
    min-height: 44px;
    font-size: 1rem;
  }

  .name-row {
    grid-template-columns: 1fr;
  }
}
</style>
