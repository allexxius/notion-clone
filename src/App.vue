<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { EditorContent, useEditor } from '@tiptap/vue-3'
import { BubbleMenu } from '@tiptap/vue-3/menus'
import StarterKit from '@tiptap/starter-kit'
import Placeholder from '@tiptap/extension-placeholder'
import DragHandle from '@tiptap/extension-drag-handle'
import TaskList from '@tiptap/extension-task-list'
import TaskItem from '@tiptap/extension-task-item'
import Link from '@tiptap/extension-link'
import { TextStyle } from '@tiptap/extension-text-style'
import { Color } from '@tiptap/extension-color'
import Highlight from '@tiptap/extension-highlight'
import {
  Bold, ChevronLeft, ChevronRight, Code2, Heading1, Heading2, Italic, Link2, List, ListOrdered, ListTodo, Palette,
  Menu, MoreHorizontal, Quote, Redo2, Search, Share2, Strikethrough, Undo2, X
} from 'lucide-vue-next'
import Sidebar from './components/Sidebar.vue'

const name = ref('')
const email = ref('')
const error = ref('')
const profile = ref(null)
const signedIn = computed(() => profile.value !== null)
const sidebarOpen = ref(true)
const activePageId = ref(1)
const viewMode = ref('page')
const calendarDate = ref(new Date())
const outlineOpen = ref(true)
const outlineItems = ref([])
const pages = ref([])
const trashedPages = ref([])
const trashOpen = ref(false)
const saveState = ref('saved')
const iconPickerOpen = ref(false)
const coverPickerOpen = ref(false)
const pageMenuOpen = ref(false)
const shareOpen = ref(false)
const shareCopied = ref(false)
const shareMessage = ref('')
const shareLinkInput = ref(null)
const settingsOpen = ref(false)
const settingsName = ref('')
const settingsEmail = ref('')
const settingsError = ref('')
const helpOpen = ref(false)
const searchOpen = ref(false)
const searchQuery = ref('')
const searchInput = ref(null)
const slashMenuOpen = ref(false)
const slashQuery = ref('')
const slashActiveIndex = ref(0)
const linkOpen = ref(false)
const linkUrl = ref('')
const colorOpen = ref(false)
const colorMode = ref('text')
const pageIcons = ['👋', '📄', '📚', '🎯', '💡', '✅', '🌱', '🚀', '⭐', '📝', '🎨', '🧠']
const pageCovers = [
  { id: 'mist', label: 'Soft mist', className: 'cover-mist' },
  { id: 'peach', label: 'Warm peach', className: 'cover-peach' },
  { id: 'meadow', label: 'Green meadow', className: 'cover-meadow' },
  { id: 'sky', label: 'Clear sky', className: 'cover-sky' },
  { id: 'lavender', label: 'Lavender', className: 'cover-lavender' },
  { id: 'sand', label: 'Warm sand', className: 'cover-sand' }
]
let saveTimer = null
let lastSavedPages = ''

const starterPages = [
  { id: 1, title: 'Getting Started', icon: '👋', favorite: true, content: '', searchText: '', cover: '', status: 'Not started', dueDate: '' },
  { id: 2, title: 'My Projects', icon: '📁', favorite: false, content: '', searchText: '', cover: '', status: 'In progress', dueDate: '' },
  { id: 3, title: 'Personal Notes', icon: '📝', favorite: false, content: '', searchText: '', cover: '', status: 'Done', dueDate: '' }
]

const inboxItems = computed(() => [...pages.value]
  .sort((a, b) => (b.updatedAt || b.id) - (a.updatedAt || a.id))
  .slice(0, 8))

const calendarDays = computed(() => {
  const year = calendarDate.value.getFullYear()
  const month = calendarDate.value.getMonth()
  const firstDay = new Date(year, month, 1)
  const gridStart = new Date(year, month, 1 - firstDay.getDay())
  return Array.from({ length: 42 }, (_, index) => {
    const date = new Date(gridStart)
    date.setDate(gridStart.getDate() + index)
    const dateKey = `${date.getFullYear()}-${date.getMonth()}-${date.getDate()}`
    return {
      date,
      inMonth: date.getMonth() === month,
      pages: pages.value.filter(page => {
        if (!page.updatedAt) return false
        const updated = new Date(page.updatedAt)
          const updatedMatches = `${updated.getFullYear()}-${updated.getMonth()}-${updated.getDate()}` === dateKey
        let dueMatches = false
        if (page.dueDate) {
          const [dueYear, dueMonth, dueDay] = page.dueDate.split('-').map(Number)
          dueMatches = `${dueYear}-${dueMonth - 1}-${dueDay}` === dateKey
        }
        return updatedMatches || dueMatches
      })
    }
  })
})

const calendarLabel = computed(() => calendarDate.value.toLocaleDateString(undefined, {
  month: 'long',
  year: 'numeric'
}))

const slashCommands = [
  { id: 'heading1', label: 'Heading 1', keywords: 'h1 title', icon: Heading1 },
  { id: 'heading2', label: 'Heading 2', keywords: 'h2 subtitle', icon: Heading2 },
  { id: 'bulletList', label: 'Bulleted list', keywords: 'bullet list ul', icon: List },
  { id: 'orderedList', label: 'Numbered list', keywords: 'numbered ordered list ol', icon: ListOrdered },
  { id: 'taskList', label: 'To-do list', keywords: 'todo checkbox task', icon: ListTodo },
  { id: 'blockquote', label: 'Quote', keywords: 'quote callout', icon: Quote },
  { id: 'codeBlock', label: 'Code block', keywords: 'code', icon: Code2 }
]

const currentPage = computed(() => pages.value.find(page => page.id === activePageId.value) || pages.value[0])
const pageLink = computed(() => {
  if (!currentPage.value) return ''
  const url = new URL(window.location.href)
  url.hash = `page=${currentPage.value.id}`
  return url.toString()
})

const editor = useEditor({
  content: '',
  extensions: [
    StarterKit,
    Placeholder.configure({ placeholder: 'Start writing...' }),
    TaskList,
    TaskItem.configure({ nested: true }),
    Link.configure({ openOnClick: false, autolink: true, linkOnPaste: true }),
    TextStyle,
    Color,
    Highlight.configure({ multicolor: true }),
    DragHandle.configure({
      nested: true,
      render: () => {
        const handle = document.createElement('button')
        handle.type = 'button'
        handle.className = 'block-drag-handle'
        handle.setAttribute('aria-label', 'Drag block')
        handle.innerHTML = '<span></span><span></span><span></span><span></span><span></span><span></span>'
        return handle
      }
    })
  ],
  onUpdate: ({ editor: activeEditor }) => {
    if (!currentPage.value) return
    currentPage.value.content = activeEditor.getHTML()
    currentPage.value.searchText = activeEditor.getText()
    currentPage.value.contentFormat = 'html'
    updateOutline(activeEditor)
    currentPage.value.updatedAt = Date.now()
    updateSlashMenu(activeEditor)
  },
  onSelectionUpdate: ({ editor: activeEditor }) => {
    updateSlashMenu(activeEditor)
  }
})

const filteredSlashCommands = computed(() => {
  const query = slashQuery.value.toLocaleLowerCase()
  return slashCommands.filter(command =>
    `${command.label} ${command.keywords}`.toLocaleLowerCase().includes(query)
  )
})

const searchResults = computed(() => {
  const query = searchQuery.value.trim().toLocaleLowerCase()
  if (!query) return pages.value

  return pages.value.filter(page =>
    `${page.title} ${page.searchText || ''}`.toLocaleLowerCase().includes(query)
  )
})

watch(currentPage, page => {
  if (!editor.value) return
  editor.value.commands.setContent(page?.content || '', { emitUpdate: false })
  nextTick(() => updateOutline(editor.value))
}, { flush: 'post' })

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
        pages.value = parsedPages.map(normalizePage)
      }
    } catch {
      localStorage.removeItem('notion-clone-pages')
    }
  }
  if (!pages.value.length) pages.value = starterPages.map(normalizePage)

  const savedTrash = localStorage.getItem('notion-clone-trash')
  if (savedTrash) {
    try {
      const parsedTrash = JSON.parse(savedTrash)
      if (Array.isArray(parsedTrash)) trashedPages.value = parsedTrash.map(normalizePage)
    } catch {
      localStorage.removeItem('notion-clone-trash')
    }
  }

  const savedActivePage = Number(localStorage.getItem('notion-clone-active-page'))
  const linkedPageId = Number(new URLSearchParams(window.location.hash.slice(1)).get('page'))
  const requestedPageId = pages.value.some(page => page.id === linkedPageId) ? linkedPageId : savedActivePage
  if (pages.value.some(page => page.id === requestedPageId)) activePageId.value = requestedPageId
  lastSavedPages = JSON.stringify(pages.value)
  window.addEventListener('keydown', handleWorkspaceKeydown)
  window.addEventListener('beforeunload', persistPages)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleWorkspaceKeydown)
  window.removeEventListener('beforeunload', persistPages)
  persistPages()
})

watch(pages, schedulePageSave, { deep: true })
watch(trashedPages, value => {
  localStorage.setItem('notion-clone-trash', JSON.stringify(value))
}, { deep: true })
watch(activePageId, value => {
  localStorage.setItem('notion-clone-active-page', String(value))
  const url = new URL(window.location.href)
  url.hash = `page=${value}`
  window.history.replaceState(null, '', url)
})

const saveStatusText = computed(() => {
  if (saveState.value === 'saving') return 'Saving...'
  if (saveState.value === 'error') return "Couldn't save"
  return 'Saved'
})

function normalizePage(page) {
  const content = typeof page.content === 'string' ? page.content : ''
  const hasHtmlContent = page.contentFormat === 'html'
  return {
    ...page,
    content: hasHtmlContent ? content : plainTextToHtml(content),
    contentFormat: 'html',
    searchText: typeof page.searchText === 'string' ? page.searchText : content,
    cover: typeof page.cover === 'string' ? page.cover : '',
    status: typeof page.status === 'string' ? page.status : 'Not started',
    dueDate: typeof page.dueDate === 'string' ? page.dueDate : '',
    updatedAt: Number(page.updatedAt) || Number(page.id) || Date.now()
  }
}

function schedulePageSave(value) {
  const serializedPages = JSON.stringify(value)
  if (serializedPages === lastSavedPages) return

  saveState.value = 'saving'
  if (saveTimer) window.clearTimeout(saveTimer)
  saveTimer = window.setTimeout(persistPages, 300)
}

function persistPages() {
  if (saveTimer) {
    window.clearTimeout(saveTimer)
    saveTimer = null
  }

  const serializedPages = JSON.stringify(pages.value)
  if (serializedPages === lastSavedPages) {
    saveState.value = 'saved'
    return
  }

  try {
    localStorage.setItem('notion-clone-pages', serializedPages)
    lastSavedPages = serializedPages
    saveState.value = 'saved'
  } catch {
    saveState.value = 'error'
  }
}

function updateOutline(activeEditor) {
  if (!activeEditor) return
  const items = []
  activeEditor.state.doc.descendants((node, pos) => {
    if (node.type.name !== 'heading') return
    const level = node.attrs.level
    const text = node.textContent.trim()
    if (!text) return
    const id = `heading-${items.length + 1}`
    items.push({ id, level, text, pos })
  })
  outlineItems.value = items
}

function scrollToHeading(item) {
  if (!editor.value) return
  const headings = [...document.querySelectorAll('.page-editor .tiptap h1, .page-editor .tiptap h2')]
  const heading = headings.find(element => element.textContent.trim() === item.text)
  if (heading) {
    heading.scrollIntoView({ behavior: 'smooth', block: 'start' })
    return
  }

  editor.value.chain().focus().setTextSelection(item.pos + 1).run()
  nextTick(() => {
    const pageEditor = document.querySelector('.page-editor')
    pageEditor?.scrollIntoView({ behavior: 'smooth', block: 'start' })
  })
}

function plainTextToHtml(text) {
  if (!text) return ''
  return text.split(/\r?\n/).map(line => `<p>${escapeHtml(line) || '<br>'}</p>`).join('')
}

function escapeHtml(text) {
  return text.replace(/[&<>"']/g, character => ({
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#39;'
  })[character])
}

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
  viewMode.value = 'page'
  searchOpen.value = false
}

function openHome() {
  viewMode.value = 'home'
}

function openInbox() {
  viewMode.value = 'inbox'
}

function openCalendar() {
  viewMode.value = 'calendar'
}

function moveCalendarMonth(direction) {
  const date = new Date(calendarDate.value)
  date.setMonth(date.getMonth() + direction)
  calendarDate.value = date
}

function reorderPages({ sourceId, targetId, after }) {
  if (sourceId === targetId) return
  const reorderedPages = [...pages.value]
  const sourceIndex = reorderedPages.findIndex(page => page.id === sourceId)
  const targetIndex = reorderedPages.findIndex(page => page.id === targetId)
  if (sourceIndex < 0 || targetIndex < 0) return

  const [page] = reorderedPages.splice(sourceIndex, 1)
  let insertIndex = targetIndex + (after ? 1 : 0)
  if (sourceIndex < insertIndex) insertIndex -= 1
  reorderedPages.splice(insertIndex, 0, page)
  pages.value = reorderedPages
}

function movePage(pageId, direction) {
  const reorderedPages = [...pages.value]
  const sourceIndex = reorderedPages.findIndex(page => page.id === pageId)
  const targetIndex = sourceIndex + direction
  if (sourceIndex < 0 || targetIndex < 0 || targetIndex >= reorderedPages.length) return

  const movedPage = reorderedPages[sourceIndex]
  reorderedPages[sourceIndex] = reorderedPages[targetIndex]
  reorderedPages[targetIndex] = movedPage
  pages.value = reorderedPages
}

function openSearch() {
  searchQuery.value = ''
  searchOpen.value = true
  nextTick(() => searchInput.value?.focus())
}

function openHelp() {
  helpOpen.value = true
}

function resetWorkspace() {
  if (!window.confirm('Reset this workspace? Your profile, pages, and trash will be deleted from this browser.')) return
  localStorage.removeItem('notion-clone-profile')
  localStorage.removeItem('notion-clone-pages')
  localStorage.removeItem('notion-clone-trash')
  localStorage.removeItem('notion-clone-active-page')
  window.location.hash = ''
  window.location.reload()
}

function openSettings() {
  settingsName.value = profile.value?.name || ''
  settingsEmail.value = profile.value?.email || ''
  settingsError.value = ''
  settingsOpen.value = true
}

function saveSettings() {
  const displayName = settingsName.value.trim()
  if (!displayName) {
    settingsError.value = 'Enter a name to continue.'
    return
  }

  const updatedProfile = { name: displayName, email: settingsEmail.value.trim() }
  try {
    localStorage.setItem('notion-clone-profile', JSON.stringify(updatedProfile))
    profile.value = updatedProfile
    settingsOpen.value = false
  } catch {
    settingsError.value = 'Your changes could not be saved in this browser.'
  }
}

function openShare() {
  shareCopied.value = false
  shareMessage.value = ''
  shareOpen.value = true
}

async function copyPageLink() {
  try {
    await navigator.clipboard.writeText(pageLink.value)
    shareCopied.value = true
    shareMessage.value = 'Link copied. It opens this page in this browser only.'
  } catch {
    shareLinkInput.value?.select()
    shareMessage.value = 'Copy the selected link. Page data is stored only in this browser.'
  }
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
  const now = Date.now()
  const page = { id: now, title: 'Untitled', icon: '📄', favorite: false, content: '', searchText: '', contentFormat: 'html', cover: '', status: 'Not started', dueDate: '', updatedAt: now }
  pages.value.push(page)
  activePageId.value = page.id
  viewMode.value = 'page'
  iconPickerOpen.value = false
  pageMenuOpen.value = false
}

function openLinkPopover() {
  linkUrl.value = editor.value?.getAttributes('link').href || ''
  linkOpen.value = !linkOpen.value
}

function applyColor(color) {
  if (!editor.value) return
  if (colorMode.value === 'highlight') {
    editor.value.chain().focus().toggleHighlight({ color }).run()
  } else {
    editor.value.chain().focus().setColor(color).run()
  }
}

function removeColorFormatting() {
  if (!editor.value) return
  editor.value.chain().focus().unsetColor().unsetHighlight().run()
  colorOpen.value = false
}

function applyLink() {
  if (!editor.value) return
  const rawUrl = linkUrl.value.trim()
  if (!rawUrl) {
    editor.value.chain().focus().unsetLink().run()
  } else {
    const href = /^https?:\/\//i.test(rawUrl) ? rawUrl : `https://${rawUrl}`
    editor.value.chain().focus().extendMarkRange('link').setLink({ href }).run()
  }
  linkOpen.value = false
}

function runEditorCommand(command) {
  if (!editor.value) return
  command(editor.value.chain().focus()).run()
}

function updateSlashMenu(activeEditor) {
  const { $from } = activeEditor.state.selection
  const textBeforeCursor = $from.parent.textBetween(0, $from.parentOffset)
  const match = textBeforeCursor.match(/^\/([a-z0-9]*)$/i)

  if (!match) {
    slashMenuOpen.value = false
    slashQuery.value = ''
    slashActiveIndex.value = 0
    return
  }

  if (!slashMenuOpen.value || slashQuery.value !== match[1]) slashActiveIndex.value = 0
  slashMenuOpen.value = true
  slashQuery.value = match[1]
}

function handleEditorKeydown(event) {
  if (!slashMenuOpen.value) return

  if (event.key === 'ArrowDown' && filteredSlashCommands.value.length) {
    event.preventDefault()
    slashActiveIndex.value = (slashActiveIndex.value + 1) % filteredSlashCommands.value.length
  } else if (event.key === 'ArrowUp' && filteredSlashCommands.value.length) {
    event.preventDefault()
    slashActiveIndex.value = (slashActiveIndex.value - 1 + filteredSlashCommands.value.length) % filteredSlashCommands.value.length
  } else if (event.key === 'Enter' && filteredSlashCommands.value.length) {
    event.preventDefault()
    applySlashCommand(filteredSlashCommands.value[slashActiveIndex.value])
  } else if (event.key === 'Escape') {
    event.preventDefault()
    slashMenuOpen.value = false
  }
}

function applySlashCommand(command) {
  if (!editor.value || !command) return
  const { $from } = editor.value.state.selection
  const triggerRange = { from: $from.start(), to: $from.pos }
  const chain = editor.value.chain().focus().deleteRange(triggerRange)

  const actions = {
    heading1: () => chain.toggleHeading({ level: 1 }).run(),
    heading2: () => chain.toggleHeading({ level: 2 }).run(),
    bulletList: () => chain.toggleBulletList().run(),
    orderedList: () => chain.toggleOrderedList().run(),
    taskList: () => chain.toggleTaskList().run(),
    blockquote: () => chain.toggleBlockquote().run(),
    codeBlock: () => chain.toggleCodeBlock().run()
  }

  actions[command.id]?.()
  slashMenuOpen.value = false
  slashQuery.value = ''
}

function setPageCover(cover) {
  if (currentPage.value) currentPage.value.cover = cover
  coverPickerOpen.value = false
}

function setPageIcon(icon) {
  if (currentPage.value) currentPage.value.icon = icon
  iconPickerOpen.value = false
}

function duplicateCurrentPage() {
  if (!currentPage.value) return
  const now = Date.now()
  const copy = {
    ...currentPage.value,
    id: now,
    title: `${currentPage.value.title || 'Untitled'} copy`,
    updatedAt: now,
    deletedAt: undefined
  }
  delete copy.deletedAt
  pages.value.push(copy)
  activePageId.value = copy.id
  viewMode.value = 'page'
  pageMenuOpen.value = false
}

function toggleFavorite() {
  if (currentPage.value) currentPage.value.favorite = !currentPage.value.favorite
  pageMenuOpen.value = false
}

function deleteCurrentPage() {
  if (pages.value.length <= 1 || !currentPage.value) return
  const confirmed = window.confirm(`Move “${currentPage.value.title}” to Trash?`)
  if (!confirmed) return

  const deletedPage = { ...currentPage.value, deletedAt: Date.now() }
  pages.value = pages.value.filter(page => page.id !== activePageId.value)
  trashedPages.value.unshift(deletedPage)
  activePageId.value = pages.value[0].id
  pageMenuOpen.value = false
  iconPickerOpen.value = false
}

function restorePage(pageId) {
  const pageIndex = trashedPages.value.findIndex(page => page.id === pageId)
  if (pageIndex < 0) return
  const [page] = trashedPages.value.splice(pageIndex, 1)
  delete page.deletedAt
  pages.value.push(page)
  activePageId.value = page.id
  viewMode.value = 'page'
  trashOpen.value = false
}

function permanentlyDeletePage(pageId) {
  const page = trashedPages.value.find(item => item.id === pageId)
  if (!page) return
  if (!window.confirm(`Permanently delete “${page.title}”? This cannot be undone.`)) return
  trashedPages.value = trashedPages.value.filter(item => item.id !== pageId)
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
      @reorder-pages="reorderPages"
      @move-page="movePage"
      @open-search="openSearch"
      @open-settings="openSettings"
      @open-trash="trashOpen = true"
      @open-help="openHelp"
      @open-home="openHome"
      @open-inbox="openInbox"
      @open-calendar="openCalendar"
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
          <span class="breadcrumb-current">{{ viewMode === 'page' ? currentPage?.title : viewMode.charAt(0).toUpperCase() + viewMode.slice(1) }}</span>
        </div>
        <div class="topbar-actions">
          <span class="updated-text" role="status" aria-live="polite">{{ saveStatusText }}</span>
          <button v-if="viewMode === 'page'" class="topbar-icon" type="button" title="Toggle page outline" :aria-pressed="outlineOpen" @click="outlineOpen = !outlineOpen"><List :size="16" /></button>
          <button class="share-button" type="button" @click="openShare"><Share2 :size="15" /> Share</button>
          <div class="page-menu-wrap">
            <button class="topbar-icon" type="button" title="More options" @click="pageMenuOpen = !pageMenuOpen">
              <MoreHorizontal :size="19" />
            </button>
            <div v-if="pageMenuOpen" class="page-menu">
              <button type="button" @click="duplicateCurrentPage">Duplicate</button>
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

      <div v-if="viewMode === 'home'" class="home-container">
        <section class="home-view">
          <p class="home-eyebrow">Workspace</p>
          <h1>Home</h1>
          <button type="button" class="home-new-page" @click="createPage"><span>+</span> New page</button>

          <section class="home-section">
            <h2>Favorites</h2>
            <div class="home-grid">
              <button v-for="page in pages.filter(item => item.favorite)" :key="page.id" type="button" class="home-page-card" @click="selectPage(page.id)">
                <span class="home-card-icon">{{ page.icon || '📄' }}</span>
                <strong>{{ page.title || 'Untitled' }}</strong>
              </button>
              <p v-if="!pages.some(item => item.favorite)" class="home-empty">No favorite pages yet.</p>
            </div>
          </section>

          <section class="home-section">
            <h2>Recent pages</h2>
            <div class="home-grid">
              <button v-for="page in pages" :key="page.id" type="button" class="home-page-card" @click="selectPage(page.id)">
                <span class="home-card-icon">{{ page.icon || '📄' }}</span>
                <strong>{{ page.title || 'Untitled' }}</strong>
              </button>
            </div>
          </section>
        </section>
      </div>

      <div v-else-if="viewMode === 'inbox'" class="utility-container">
        <section class="utility-view">
          <p class="home-eyebrow">Workspace</p>
          <h1>Inbox</h1>
          <p class="utility-subtitle">Recent changes in your workspace</p>
          <div class="inbox-list">
            <button v-for="page in inboxItems" :key="page.id" type="button" class="inbox-row" @click="selectPage(page.id)">
              <span class="inbox-icon">{{ page.icon || '📄' }}</span>
              <span class="inbox-copy">
                <strong>{{ page.title || 'Untitled' }}</strong>
                <small>Updated {{ new Date(page.updatedAt).toLocaleString() }}</small>
              </span>
              <ChevronRight :size="15" />
            </button>
            <p v-if="!inboxItems.length" class="home-empty">No recent activity.</p>
          </div>
        </section>
      </div>

      <div v-else-if="viewMode === 'calendar'" class="utility-container">
        <section class="utility-view calendar-view">
          <p class="home-eyebrow">Workspace</p>
          <header class="calendar-header">
            <h1>{{ calendarLabel }}</h1>
            <div class="calendar-controls">
              <button type="button" aria-label="Previous month" @click="moveCalendarMonth(-1)"><ChevronLeft :size="16" /></button>
              <button type="button" aria-label="Next month" @click="moveCalendarMonth(1)"><ChevronRight :size="16" /></button>
            </div>
          </header>
          <div class="calendar-weekdays"><span v-for="day in ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']" :key="day">{{ day }}</span></div>
          <div class="calendar-grid">
            <button
              v-for="day in calendarDays"
              :key="day.date.toISOString()"
              type="button"
              class="calendar-day"
              :class="{ 'outside-month': !day.inMonth }"
              @click="day.pages.length === 1 && selectPage(day.pages[0].id)"
            >
              <span class="calendar-date">{{ day.date.getDate() }}</span>
              <span v-for="page in day.pages.slice(0, 2)" :key="page.id" class="calendar-page" @click.stop="selectPage(page.id)">{{ page.icon }} {{ page.title }}</span>
            </button>
          </div>
        </section>
      </div>

      <div v-else-if="currentPage" class="page-container" :class="{ 'with-outline': outlineOpen }">
        <aside v-if="outlineOpen && outlineItems.length" class="page-outline">
          <p>On this page</p>
          <button
            v-for="item in outlineItems"
            :key="item.id"
            type="button"
            :class="`outline-level-${item.level}`"
            @click="scrollToHeading(item)"
          >{{ item.text }}</button>
        </aside>
        <article class="page-content" :class="{ 'has-cover': currentPage.cover }">
          <div v-if="currentPage.cover" :class="['page-cover', currentPage.cover]">
            <button type="button" class="page-cover-change" @click="coverPickerOpen = !coverPickerOpen">Change cover</button>
          </div>
          <div v-else class="page-cover-control">
            <button type="button" @click="coverPickerOpen = !coverPickerOpen">Add cover</button>
          </div>
          <div v-if="coverPickerOpen" class="cover-picker">
            <div class="icon-picker-heading">Choose a cover</div>
            <div class="cover-grid">
              <button v-for="cover in pageCovers" :key="cover.id" type="button" :class="cover.className" :title="cover.label" :aria-label="cover.label" @click="setPageCover(cover.className)"></button>
            </div>
            <button v-if="currentPage.cover" type="button" class="remove-icon" @click="setPageCover('')">Remove cover</button>
          </div>
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
          <div class="page-properties">
            <label>
              <span>Status</span>
              <select v-model="currentPage.status">
                <option>Not started</option>
                <option>In progress</option>
                <option>Done</option>
              </select>
            </label>
            <label>
              <span>Due date</span>
              <input v-model="currentPage.dueDate" type="date" aria-label="Due date" />
            </label>
          </div>
          <div class="editor-toolbar" role="toolbar" aria-label="Text formatting">
            <button type="button" title="Bold" aria-label="Bold" :class="{ active: editor?.isActive('bold') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleBold())"><Bold :size="16" /></button>
            <button type="button" title="Italic" aria-label="Italic" :class="{ active: editor?.isActive('italic') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleItalic())"><Italic :size="16" /></button>
            <div class="color-control">
              <button type="button" title="Text color" aria-label="Text color" :class="{ active: colorOpen }" @mousedown.prevent @click="colorOpen = !colorOpen"><Palette :size="16" /></button>
              <div v-if="colorOpen" class="color-popover">
                <div class="color-tabs">
                  <button type="button" :class="{ active: colorMode === 'text' }" @click="colorMode = 'text'">Text</button>
                  <button type="button" :class="{ active: colorMode === 'highlight' }" @click="colorMode = 'highlight'">Highlight</button>
                </div>
                <div class="color-grid">
                  <button v-for="color in ['#37352f', '#787774', '#d44c47', '#d9730d', '#448361', '#0b6e99', '#6940a5', '#ad1a72']" :key="color" type="button" :title="color" :aria-label="`Apply ${color}`" :style="{ background: color }" @mousedown.prevent @click="applyColor(color)"></button>
                </div>
                <button type="button" class="remove-formatting" @click="removeColorFormatting">Remove color</button>
              </div>
            </div>
            <div class="link-control">
              <button type="button" title="Add link" aria-label="Add link" :class="{ active: editor?.isActive('link') }" @mousedown.prevent @click="openLinkPopover"><Link2 :size="16" /></button>
              <div v-if="linkOpen" class="link-popover">
                <input v-model="linkUrl" type="url" placeholder="https://example.com" aria-label="Link destination" @keydown.enter.prevent="applyLink" />
                <div class="link-actions">
                  <button v-if="editor?.isActive('link')" type="button" class="link-remove" @click="linkUrl = ''; applyLink">Remove</button>
                  <button type="button" class="link-apply" @click="applyLink">Apply</button>
                </div>
              </div>
            </div>
            <span class="toolbar-divider"></span>
            <button type="button" title="Heading 1" aria-label="Heading 1" :class="{ active: editor?.isActive('heading', { level: 1 }) }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleHeading({ level: 1 }))"><Heading1 :size="17" /></button>
            <button type="button" title="Heading 2" aria-label="Heading 2" :class="{ active: editor?.isActive('heading', { level: 2 }) }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleHeading({ level: 2 }))"><Heading2 :size="17" /></button>
            <span class="toolbar-divider"></span>
            <button type="button" title="Bulleted list" aria-label="Bulleted list" :class="{ active: editor?.isActive('bulletList') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleBulletList())"><List :size="17" /></button>
            <button type="button" title="Numbered list" aria-label="Numbered list" :class="{ active: editor?.isActive('orderedList') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleOrderedList())"><ListOrdered :size="17" /></button>
            <button type="button" title="To-do list" aria-label="To-do list" :class="{ active: editor?.isActive('taskList') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleTaskList())"><ListTodo :size="17" /></button>
            <button type="button" title="Quote" aria-label="Quote" :class="{ active: editor?.isActive('blockquote') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleBlockquote())"><Quote :size="16" /></button>
            <span class="toolbar-spacer"></span>
            <button type="button" title="Undo" aria-label="Undo" :disabled="!editor?.can().undo()" @mousedown.prevent @click="runEditorCommand(chain => chain.undo())"><Undo2 :size="16" /></button>
            <button type="button" title="Redo" aria-label="Redo" :disabled="!editor?.can().redo()" @mousedown.prevent @click="runEditorCommand(chain => chain.redo())"><Redo2 :size="16" /></button>
          </div>
          <div class="editor-wrap" @keydown="handleEditorKeydown">
            <BubbleMenu :editor="editor" :tippy-options="{ duration: 100, placement: 'top' }" class="selection-bubble-menu">
              <button type="button" title="Bold" aria-label="Bold" :class="{ active: editor?.isActive('bold') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleBold())"><Bold :size="15" /></button>
              <button type="button" title="Italic" aria-label="Italic" :class="{ active: editor?.isActive('italic') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleItalic())"><Italic :size="15" /></button>
              <button type="button" title="Strikethrough" aria-label="Strikethrough" :class="{ active: editor?.isActive('strike') }" @mousedown.prevent @click="runEditorCommand(chain => chain.toggleStrike())"><Strikethrough :size="15" /></button>
              <span class="bubble-divider"></span>
              <button type="button" title="Clear formatting" aria-label="Clear formatting" @mousedown.prevent @click="runEditorCommand(chain => chain.unsetAllMarks())"><X :size="15" /></button>
            </BubbleMenu>
            <div v-if="slashMenuOpen" class="slash-menu" role="listbox" aria-label="Block commands">
              <p class="slash-menu-label">Insert a block</p>
              <button
                v-for="(command, index) in filteredSlashCommands"
                :key="command.id"
                type="button"
                role="option"
                :aria-selected="index === slashActiveIndex"
                :class="{ active: index === slashActiveIndex }"
                @mousedown.prevent
                @click="applySlashCommand(command)"
              >
                <span class="slash-command-icon"><component :is="command.icon" :size="17" /></span>
                <span>{{ command.label }}</span>
              </button>
              <p v-if="!filteredSlashCommands.length" class="slash-menu-empty">No matching blocks</p>
            </div>
            <EditorContent :editor="editor" class="page-editor" aria-label="Page content" />
          </div>
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

    <div v-if="trashOpen" class="share-backdrop" @click.self="trashOpen = false">
      <section class="share-dialog trash-dialog" role="dialog" aria-modal="true" aria-labelledby="trash-heading">
        <header class="share-dialog-header">
          <h2 id="trash-heading">Trash</h2>
          <button type="button" aria-label="Close trash" @click="trashOpen = false"><X :size="17" /></button>
        </header>
        <p v-if="!trashedPages.length" class="trash-empty">Trash is empty.</p>
        <div v-else class="trash-list">
          <article v-for="page in trashedPages" :key="page.id" class="trash-row">
            <span class="trash-page-icon">{{ page.icon || '📄' }}</span>
            <div class="trash-page-name">
              <strong>{{ page.title || 'Untitled' }}</strong>
              <small>Deleted {{ new Date(page.deletedAt).toLocaleDateString() }}</small>
            </div>
            <button type="button" class="trash-restore" @click="restorePage(page.id)">Restore</button>
            <button type="button" class="trash-delete" aria-label="Permanently delete page" @click="permanentlyDeletePage(page.id)">Delete</button>
          </article>
        </div>
      </section>
    </div>

    <div v-if="shareOpen" class="share-backdrop" @click.self="shareOpen = false">
      <section class="share-dialog" role="dialog" aria-modal="true" aria-labelledby="share-heading">
        <header class="share-dialog-header">
          <h2 id="share-heading">Share “{{ currentPage?.title || 'Untitled' }}”</h2>
          <button type="button" aria-label="Close share dialog" @click="shareOpen = false"><X :size="17" /></button>
        </header>
        <p class="share-notice">This workspace is saved in this browser. The link opens this page here, but it does not share or sync your content to other devices.</p>
        <label class="share-link-label" for="page-link">Page link</label>
        <div class="share-link-row">
          <input id="page-link" ref="shareLinkInput" :value="pageLink" readonly />
          <button type="button" @click="copyPageLink">{{ shareCopied ? 'Copied' : 'Copy link' }}</button>
        </div>
        <p v-if="shareMessage" class="share-message" aria-live="polite">{{ shareMessage }}</p>
      </section>
    </div>

    <div v-if="helpOpen" class="share-backdrop" @click.self="helpOpen = false">
      <section class="share-dialog help-dialog" role="dialog" aria-modal="true" aria-labelledby="help-heading">
        <header class="share-dialog-header">
          <h2 id="help-heading">Help & support</h2>
          <button type="button" aria-label="Close help" @click="helpOpen = false"><X :size="17" /></button>
        </header>
        <div class="help-section">
          <h3>Keyboard shortcuts</h3>
          <div class="shortcut-row"><span>Search workspace</span><kbd>Ctrl K</kbd></div>
          <div class="shortcut-row"><span>Move page up or down</span><kbd>Alt ↑ / ↓</kbd></div>
          <div class="shortcut-row"><span>Close a dialog</span><kbd>Esc</kbd></div>
        </div>
        <div class="help-section">
          <h3>Writing tips</h3>
          <p>Type <kbd>/</kbd> in a paragraph to insert a heading, list, quote, or code block.</p>
          <p>Use the formatting toolbar for bold, italic, lists, undo, and redo.</p>
        </div>
        <div class="help-reset">
          <button type="button" class="trash-delete" @click="resetWorkspace">Reset local workspace</button>
        </div>
      </section>
    </div>

    <div v-if="settingsOpen" class="share-backdrop" @click.self="settingsOpen = false">
      <section class="share-dialog settings-dialog" role="dialog" aria-modal="true" aria-labelledby="settings-heading">
        <header class="share-dialog-header">
          <h2 id="settings-heading">Profile settings</h2>
          <button type="button" aria-label="Close settings" @click="settingsOpen = false"><X :size="17" /></button>
        </header>
        <p class="share-notice">Update the name and email shown in your local workspace.</p>
        <form class="settings-form" @submit.prevent="saveSettings">
          <label for="settings-name">Name</label>
          <input id="settings-name" v-model="settingsName" required autocomplete="name" />
          <label for="settings-email">Email <span class="optional">Optional</span></label>
          <input id="settings-email" v-model="settingsEmail" type="email" autocomplete="email" />
          <p v-if="settingsError" class="error-message" role="alert">{{ settingsError }}</p>
          <div class="settings-actions">
            <button type="button" class="settings-cancel" @click="settingsOpen = false">Cancel</button>
            <button type="submit" class="settings-save">Save changes</button>
          </div>
        </form>
      </section>
    </div>
  </div>
</template>

