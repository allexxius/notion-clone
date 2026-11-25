<script setup>
import {
  Search, Settings, Plus, ChevronDown, Home, Inbox, CalendarDays,
  Star, MoreHorizontal, CircleHelp, ChevronsLeft, LogOut
} from 'lucide-vue-next'

defineProps({
  activePage: Number,
  pages: Array,
  profile: Object
})

const emit = defineEmits(['selectPage', 'createPage', 'openSearch', 'logout'])
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
      <button class="sidebar-button"><Settings :size="17" /><span>Settings</span></button>
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
        :class="{ active: activePage === page.id }"
        @click="emit('selectPage', page.id)"
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
      <button class="sidebar-button"><CircleHelp :size="17" /><span>Help & support</span></button>
      <button class="user-profile" @click="emit('logout')" title="Log out">
        <div class="user-avatar">{{ profile.name.charAt(0).toUpperCase() }}</div>
        <div class="user-info"><span>{{ profile.name }}</span><small>{{ profile.email || 'Personal workspace' }}</small></div>
        <LogOut :size="15" />
      </button>
    </div>
  </aside>
</template>
