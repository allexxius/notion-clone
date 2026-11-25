<script setup>
import { computed, onMounted, ref, watch } from 'vue'
import { ChevronRight, Menu, MoreHorizontal, Share2 } from 'lucide-vue-next'
import Sidebar from './components/Sidebar.vue'

const name = ref('')
const email = ref('')
const error = ref('')
const profile = ref(null)
const signedIn = computed(() => profile.value !== null)
const sidebarOpen = ref(true)
const activePageId = ref(1)
const pages = ref([])
const editorHeight = ref(420)

const starterPages = [
  { id: 1, title: 'Getting Started', icon: '👋', favorite: true, content: '' },
  { id: 2, title: 'My Projects', icon: '📁', favorite: false, content: '' },
  { id: 3, title: 'Personal Notes', icon: '📝', favorite: false, content: '' }
]

const currentPage = computed(() => pages.value.find(page => page.id === activePageId.value) || pages.value[0])

onMounted(() => {
  const savedProfile = localStorage.getItem('notion-clone-profile')
  if (savedProfile) {
    try {
      profile.value = JSON.parse(savedProfile)
    } catch {
      localStorage.removeItem('notion-clone-profile')
    }
  }

  const savedPages = localStorage.getItem('notion-clone-pages')
  if (savedPages) {
    try {
      const parsedPages = JSON.parse(savedPages)
      if (Array.isArray(parsedPages)) {
        pages.value = parsedPages.map(page => ({
          ...page,
          content: typeof page.content === 'string' ? page.content : ''
        }))
      }
    } catch {
      localStorage.removeItem('notion-clone-pages')
    }
  }
  if (!pages.value.length) pages.value = starterPages

  const savedActivePage = Number(localStorage.getItem('notion-clone-active-page'))
  if (pages.value.some(page => page.id === savedActivePage)) activePageId.value = savedActivePage
})

watch(pages, value => localStorage.setItem('notion-clone-pages', JSON.stringify(value)), { deep: true })
watch(activePageId, value => localStorage.setItem('notion-clone-active-page', String(value)))
watch(currentPage, () => resizeEditor(), { flush: 'post' })

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

function selectPage(id) {
  activePageId.value = id
  editorHeight.value = 420
}

function createPage() {
  const page = { id: Date.now(), title: 'Untitled', icon: '📄', favorite: false, content: '' }
  pages.value.push(page)
  activePageId.value = page.id
  editorHeight.value = 420
}

function resizeEditor(event) {
  const editor = event?.target || document.querySelector('.page-editor')
  if (!editor) return
  editor.style.height = 'auto'
  editorHeight.value = Math.max(420, editor.scrollHeight)
}
</script>

<template>
  <main v-if="!signedIn" class="login-screen">
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

  </main>

  <div v-else class="app-layout" :class="{ 'sidebar-closed': !sidebarOpen }">
    <Sidebar
      v-if="sidebarOpen"
      :pages="pages"
      :active-page="activePageId"
      :profile="profile"
      @select-page="selectPage"
      @create-page="createPage"
      @logout="logOut"
    />

    <main class="main-content">
      <header class="topbar">
        <button class="sidebar-toggle" type="button" title="Toggle sidebar" @click="sidebarOpen = !sidebarOpen">
          <Menu :size="17" />
        </button>
        <div class="breadcrumb">
          <span class="breadcrumb-icon">{{ profile.name.charAt(0).toUpperCase() }}</span>
          <span>{{ profile.name }}'s Workspace</span>
          <ChevronRight :size="14" />
          <span class="breadcrumb-current">{{ currentPage?.title }}</span>
        </div>
        <div class="topbar-actions">
          <span class="updated-text">Edited just now</span>
          <button class="share-button" type="button"><Share2 :size="15" /> Share</button>
          <button class="topbar-icon" type="button" title="More options"><MoreHorizontal :size="19" /></button>
        </div>
      </header>

      <div v-if="currentPage" class="page-container">
        <article class="page-content">
          <span class="page-icon-large">{{ currentPage.icon }}</span>
          <input
            v-model="currentPage.title"
            class="page-title-input"
            aria-label="Page title"
            placeholder="Untitled"
          />
          <textarea
            v-model="currentPage.content"
            class="page-editor"
            :style="{ height: `${editorHeight}px` }"
            placeholder="Start writing..."
            aria-label="Page content"
            @input="resizeEditor"
          ></textarea>
        </article>
      </div>
    </main>
  </div>
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

.page-title-input {
  display: block;
  width: 100%;
  height: auto;
  padding: 0 0 14px;
  border: 0;
  border-radius: 0;
  outline: 0;
  background: transparent;
  color: #37352f;
  font-family: Georgia, "Times New Roman", serif;
  font-size: 42px;
  font-weight: 700;
  line-height: 1.2;
  letter-spacing: 0;
  box-shadow: none;
}

.page-title-input:focus { border: 0; box-shadow: none; }
.page-title-input::placeholder { color: #d6d6d3; }

.page-editor {
  display: block;
  width: 100%;
  min-height: 420px;
  padding: 0;
  border: 0;
  border-radius: 0;
  outline: 0;
  overflow: hidden;
  resize: none;
  background: transparent;
  color: #37352f;
  font: inherit;
  font-size: 16px;
  line-height: 1.75;
  box-shadow: none;
}

.page-editor:focus { border: 0; box-shadow: none; }
.page-editor::placeholder { color: #b8b7b4; }

@media (max-width: 760px) {
  .login-screen { padding-inline: 22px; }
  .page-title-input { font-size: 34px; }
}

@media (max-width: 480px) {
  .brand { top: 20px; left: 20px; }
  .login-panel h1 { font-size: 30px; }
}
</style>
