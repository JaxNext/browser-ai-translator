<script setup lang="ts">
import NotSupport from '@/components/NotSupport.vue'
import DarkMode from '@/components/DarkMode.vue'
import Translator from '@/components/Translator.vue'
import { ref } from 'vue'

const isSupport = ref(true)
</script>

<template>
  <div class="dark:bg-[rgb(2,8,23)] min-h-screen box-border p-6">
    <DarkMode class="fixed right-2 top-2" />

    <Translator v-if="isSupport" />
    <NotSupport v-else />
  </div>
</template>