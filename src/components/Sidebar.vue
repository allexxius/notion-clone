<script setup>
import { ref } from 'vue'
import {
  Search, Settings, Plus, ChevronDown, Home, Inbox, CalendarDays,
  Star, MoreHorizontal, CircleHelp, LogOut
} from 'lucide-vue-next'

defineProps({
  activePage: Number,
  pages: Array,
  profile: Object
})

const emit = defineEmits(['selectPage', 'createPage', 'openSearch', 'openSettings', 'openTrash', 'logout', 'reorderPages', 'movePage'])
const draggingPageId = ref(null)
const dropTargetPageId = ref(null)

function startPageDrag(pageId, event) {
  draggingPageId.value = pageId
  dropTargetPageId.value = null
  event.dataTransfer.effectAllowed = 'move'
  event.dataTransfer.setData('text/plain', String(pageId))
}

function dropPage(targetId, event) {
  const sourceId = Number(event.dataTransfer.getData('text/plain'))
  const targetBounds = event.currentTarget.getBoundingClientRect()
  const after = event.clientY >= targetBounds.top + targetBounds.height / 2
  if (sourceId) emit('reorderPages', { sourceId, targetId, after })
  draggingPageId.value = null
  dropTargetPageId.value = null
}
</script>

<template>
  <aside class="sidebar">
    <div class="workspace">
      <div class="workspace-icon">{{ profile.name.charAt(0).toUpperCase() }}</div>
      <div class="workspace-name">
        <span>{{ profile.name }}'s Workspace</span>
        <small>Free Plan</small>
      </div>
      <ChevronDown :size="16" />
    </div>

    <div class="sidebar-navigation">
      <button class="sidebar-button" @click="emit('openSearch')">
        <Search :size="17" /><span>Search</span><span class="shortcut">Ctrl K</span>
      </button>
      <button class="sidebar-button" @click="emit('openSettings')"><Settings :size="17" /><span>Settings</span></button>
      <button class="sidebar-button"><Home :size="17" /><span>Home</span></button>
      <button class="sidebar-button"><Inbox :size="17" /><span>Inbox</span></button>
      <button class="sidebar-button"><CalendarDays :size="17" /><span>Calendar</span></button>
    </div>

    <div class="sidebar-section">
      <div class="section-heading"><span>Favorites</span></div>
      <button v-for="page in pages.filter(page => page.favorite)" :key="page.id" class="sidebar-button page-link" @click="emit('selectPage', page.id)">
        <span class="page-icon">{{ page.icon }}</span><span class="page-name">{{ page.title }}</span>
      </button>
      <button v-if="!pages.some(page => page.favorite)" class="sidebar-button"><Star :size="16" /><span>Add your favorites</span></button>
    </div>

    <div class="sidebar-section private-section">
      <div class="section-heading">
        <span>Private</span>
        <button class="small-icon-button" @click="emit('createPage')" title="Add a page"><Plus :size="16" /></button>
      </div>

      <button
        v-for="page in pages"
        :key="page.id"
        class="sidebar-button page-link"
        :class="{
          active: activePage === page.id,
          dragging: draggingPageId === page.id,
          'drop-target': dropTargetPageId === page.id
        }"
        draggable="true"
        @click="emit('selectPage', page.id)"
        @dragstart="startPageDrag(page.id, $event)"
        @dragend="draggingPageId = null; dropTargetPageId = null"
        @dragover.prevent="dropTargetPageId = page.id"
        @drop.prevent.stop="dropPage(page.id, $event)"
        @keydown.alt.up.prevent="emit('movePage', page.id, -1)"
        @keydown.alt.down.prevent="emit('movePage', page.id, 1)"
        :aria-label="`${page.title}, use Alt plus arrow keys to reorder`"
      >
        <span class="page-icon">{{ page.icon }}</span>
        <span class="page-name">{{ page.title }}</span>
        <MoreHorizontal :size="15" class="page-more" />
      </button>

      <button class="sidebar-button add-page-button" @click="emit('createPage')">
        <Plus :size="16" /><span>Add a page</span>
      </button>
    </div>

    <div class="sidebar-bottom">
      <button class="sidebar-button" @click="emit('openTrash')"><CircleHelp :size="17" /><span>Trash</span></button>
      <button class="user-profile" @click="emit('logout')" title="Log out">
        <div class="user-avatar">{{ profile.name.charAt(0).toUpperCase() }}</div>
        <div class="user-info"><span>{{ profile.name }}</span><small>{{ profile.email || 'Personal workspace' }}</small></div>
        <LogOut :size="15" />
      </button>
    </div>
  </aside>
</template>
