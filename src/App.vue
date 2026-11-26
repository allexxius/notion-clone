<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { ChevronRight, Menu, MoreHorizontal, Search, Share2, X } from 'lucide-vue-next'
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
const iconPickerOpen = ref(false)
const pageMenuOpen = ref(false)
const searchOpen = ref(false)
const searchQuery = ref('')
const searchInput = ref(null)
const pageIcons = ['👋', '📄', '📚', '🎯', '💡', '✅', '🌱', '🚀', '⭐', '📝', '🎨', '🧠']

const starterPages = [
  { id: 1, title: 'Getting Started', icon: '👋', favorite: true, content: '' },
  { id: 2, title: 'My Projects', icon: '📁', favorite: false, content: '' },
  { id: 3, title: 'Personal Notes', icon: '📝', favorite: false, content: '' }
]

const currentPage = computed(() => pages.value.find(page => page.id === activePageId.value) || pages.value[0])
const searchResults = computed(() => {
  const query = searchQuery.value.trim().toLocaleLowerCase()
  if (!query) return pages.value

  return pages.value.filter(page =>
    `${page.title} ${page.content}`.toLocaleLowerCase().includes(query)
  )
})

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
  window.addEventListener('keydown', handleWorkspaceKeydown)
})

onBeforeUnmount(() => window.removeEventListener('keydown', handleWorkspaceKeydown))

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
  searchOpen.value = false
}

function openSearch() {
  searchQuery.value = ''
  searchOpen.value = true
  nextTick(() => searchInput.value?.focus())
}

function handleWorkspaceKeydown(event) {
  if ((event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'k') {
    event.preventDefault()
    openSearch()
  } else if (event.key === 'Escape') {
    searchOpen.value = false
  }
}

function createPage() {
  const page = { id: Date.now(), title: 'Untitled', icon: '📄', favorite: false, content: '' }
  pages.value.push(page)
  activePageId.value = page.id
  editorHeight.value = 420
  iconPickerOpen.value = false
  pageMenuOpen.value = false
}

function setPageIcon(icon) {
  if (currentPage.value) currentPage.value.icon = icon
  iconPickerOpen.value = false
}

function toggleFavorite() {
  if (currentPage.value) currentPage.value.favorite = !currentPage.value.favorite
  pageMenuOpen.value = false
}

function deleteCurrentPage() {
  if (pages.value.length <= 1 || !currentPage.value) return
  const confirmed = window.confirm(`Delete “${currentPage.value.title}”?`)
  if (!confirmed) return

  pages.value = pages.value.filter(page => page.id !== activePageId.value)
  activePageId.value = pages.value[0].id
  pageMenuOpen.value = false
  iconPickerOpen.value = false
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
      @open-search="openSearch"
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
          <div class="page-menu-wrap">
            <button class="topbar-icon" type="button" title="More options" @click="pageMenuOpen = !pageMenuOpen">
              <MoreHorizontal :size="19" />
            </button>
            <div v-if="pageMenuOpen" class="page-menu">
              <button type="button" @click="toggleFavorite">
                {{ currentPage.favorite ? 'Remove from Favorites' : 'Add to Favorites' }}
              </button>
              <button type="button" class="delete-action" :disabled="pages.length <= 1" @click="deleteCurrentPage">
                Delete page
              </button>
            </div>
          </div>
        </div>
      </header>

      <div v-if="currentPage" class="page-container">
        <article class="page-content">
          <div class="page-icon-control">
            <button
              class="page-icon-large"
              type="button"
              :aria-label="currentPage.icon ? 'Change page icon' : 'Add page icon'"
              @click="iconPickerOpen = !iconPickerOpen"
            >
              {{ currentPage.icon || '+' }}
            </button>
            <div v-if="iconPickerOpen" class="icon-picker">
              <div class="icon-picker-heading">Choose an icon</div>
              <div class="icon-grid">
                <button v-for="icon in pageIcons" :key="icon" type="button" :aria-label="`Set page icon to ${icon}`" @click="setPageIcon(icon)">
                  {{ icon }}
                </button>
              </div>
              <button class="remove-icon" type="button" @click="setPageIcon('')">Remove icon</button>
            </div>
          </div>
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

    <div v-if="searchOpen" class="search-backdrop" @click.self="searchOpen = false">
      <section class="search-dialog" role="dialog" aria-modal="true" aria-label="Search workspace">
        <div class="search-field">
          <Search :size="19" aria-hidden="true" />
          <input
            ref="searchInput"
            v-model="searchQuery"
            type="search"
            :placeholder="`Search ${profile.name}'s workspace`"
            aria-label="Search pages"
          />
          <button type="button" class="search-close" aria-label="Close search" @click="searchOpen = false">
            <X :size="17" />
          </button>
        </div>

        <div class="search-result-list">
          <p class="search-section-label">Pages</p>
          <button
            v-for="page in searchResults"
            :key="page.id"
            type="button"
            class="search-result"
            @click="selectPage(page.id)"
          >
            <span class="search-result-icon">{{ page.icon || '📄' }}</span>
            <span class="search-result-copy">
              <strong>{{ page.title || 'Untitled' }}</strong>
              <small>{{ page.content || 'No additional text' }}</small>
            </span>
          </button>
          <p v-if="!searchResults.length" class="search-empty">No matching pages</p>
        </div>
      </section>
    </div>
  </div>
</template>

