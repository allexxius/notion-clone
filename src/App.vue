<script setup>
import { computed, onMounted, ref } from 'vue'

const name = ref('')
const email = ref('')
const error = ref('')
const profile = ref(null)
const signedIn = computed(() => profile.value !== null)

onMounted(() => {
  const savedProfile = localStorage.getItem('notion-clone-profile')
  if (savedProfile) {
    try {
      profile.value = JSON.parse(savedProfile)
    } catch {
      localStorage.removeItem('notion-clone-profile')
    }
  }
})

function logIn() {
  const displayName = name.value.trim()
  if (!displayName) {
    error.value = 'Please enter your name.'
    return
  }

  profile.value = { name: displayName, email: email.value.trim() }
  localStorage.setItem('notion-clone-profile', JSON.stringify(profile.value))
  error.value = ''
}

function logOut() {
  localStorage.removeItem('notion-clone-profile')
  profile.value = null
  name.value = ''
  email.value = ''
}
</script>

<template>
  <main class="login-screen">
    <a class="brand" href="#" aria-label="Notion home">
      <span class="brand-mark" aria-hidden="true">N</span>
      <span>Notion</span>
    </a>

    <section v-if="!signedIn" class="login-panel" aria-labelledby="login-title">
      <h1 id="login-title">Log in to your workspace</h1>
      <p class="subtitle">Your space for notes, plans, and everything in between.</p>

      <form @submit.prevent="logIn">
        <label for="display-name">Your name</label>
        <input
          id="display-name"
          v-model="name"
          type="text"
          name="name"
          autocomplete="name"
          placeholder="Aleks"
          required
          autofocus
        />

        <label for="email">Email <span class="optional">Optional</span></label>
        <input
          id="email"
          v-model="email"
          type="email"
          name="email"
          autocomplete="email"
          placeholder="you@example.com"
        />

        <p v-if="error" class="error-message" role="alert">{{ error }}</p>
        <button class="continue-button" type="submit">Continue</button>
      </form>

      <p class="privacy-note">Your profile is saved in this browser.</p>
    </section>

    <section v-else class="login-panel signed-in-panel" aria-live="polite">
      <span class="welcome-mark" aria-hidden="true">{{ profile.name.charAt(0).toUpperCase() }}</span>
      <h1>Welcome, {{ profile.name }}</h1>
      <p class="subtitle">Your workspace is ready.</p>
      <button class="continue-button" type="button" @click="logOut">Log out</button>
    </section>
  </main>
</template>

<style scoped>
:global(body) {
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
  background: #fff;
}

:global(#app) {
  width: 100%;
  max-width: none;
  min-height: 100vh;
  margin: 0;
  border: 0;
  display: block;
  text-align: left;
}

.login-screen {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 88px 20px 40px;
  color: #37352f;
  font-family: ui-sans-serif, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}

.brand {
  position: absolute;
  top: 24px;
  left: 28px;
  display: inline-flex;
  align-items: center;
  gap: 9px;
  color: inherit;
  font-size: 20px;
  font-weight: 700;
  text-decoration: none;
}

.brand-mark {
  width: 28px;
  height: 28px;
  display: grid;
  place-items: center;
  border: 2px solid #37352f;
  box-shadow: 1px 1px 0 #37352f;
  font-family: Georgia, "Times New Roman", serif;
}

.login-panel {
  width: min(380px, 100%);
  text-align: center;
}

.login-panel h1 {
  margin: 0 0 9px;
  color: #37352f;
  font-size: 34px;
  line-height: 1.18;
  font-weight: 650;
  letter-spacing: 0;
}

.subtitle {
  margin: 0 0 30px;
  color: #787774;
  font-size: 14px;
  line-height: 1.5;
}

form { text-align: left; }

label {
  display: flex;
  justify-content: space-between;
  margin: 16px 0 7px;
  font-size: 13px;
  font-weight: 600;
}

.optional {
  color: #9b9a97;
  font-weight: 400;
}

input {
  width: 100%;
  height: 42px;
  padding: 0 12px;
  border: 1px solid #d9d9d6;
  border-radius: 5px;
  outline: 0;
  background: #fff;
  color: #37352f;
  font: inherit;
  font-size: 14px;
  box-shadow: 0 1px 2px rgb(15 15 15 / 4%);
}

input:focus {
  border-color: #2383e2;
  box-shadow: 0 0 0 2px rgb(35 131 226 / 12%);
}

input::placeholder { color: #b8b7b4; }

.continue-button {
  width: 100%;
  height: 42px;
  margin-top: 20px;
  border: 0;
  border-radius: 5px;
  background: #2383e2;
  color: #fff;
  font: inherit;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
}

.continue-button:hover { background: #1976cc; }
.continue-button:focus-visible { outline: 2px solid #37352f; outline-offset: 2px; }

.privacy-note {
  margin: 18px 0 0;
  color: #9b9a97;
  font-size: 12px;
}

.error-message {
  margin: 8px 0 -8px;
  color: #d44c47;
  font-size: 13px;
}

.signed-in-panel { display: flex; flex-direction: column; align-items: center; }
.signed-in-panel .subtitle { margin-bottom: 4px; }
.welcome-mark {
  width: 52px;
  height: 52px;
  display: grid;
  place-items: center;
  margin-bottom: 20px;
  border-radius: 12px;
  background: #f1f1ef;
  font-size: 23px;
  font-weight: 600;
}

@media (max-width: 480px) {
  .brand { top: 20px; left: 20px; }
  .login-panel h1 { font-size: 30px; }
}
</style>
