<script setup lang="ts">
import NotSupport from '@/components/NotSupport.vue'
import DarkMode from '@/components/DarkMode.vue'
import Translator from '@/components/Translator.vue'
import { ref, onMounted } from 'vue'
import Toaster from '@/components/ui/toast/Toaster.vue'
import { checkDetectorUsability, checkTranslatorUsability } from '@rejax/browser-ai'

const isSupport = ref(false)

onMounted(async () => {
  const canDetect = await checkDetectorUsability()
  const canTranslate = await checkTranslatorUsability({
    sourceLanguage: 'en',
    targetLanguage: 'zh',
  })
  isSupport.value = canDetect.available && canTranslate.available
})
</script>

<template>
  <div class="dark:bg-[rgb(2,8,23)] min-h-screen box-border p-6">
    <DarkMode class="fixed right-2 top-2" />

    <Translator v-if="isSupport" />
    <NotSupport v-else />
  </div>
  <Toaster />
</template>