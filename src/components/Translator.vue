<template>
  <h1 class="text-3xl text-center font-bold dark:text-slate-50">浏览器内置 AI 翻译</h1>
  <div class="mt-4 flex gap-4 h-[calc(100vh-100px)]">
    <Card class="w-1/2">
      <CardContent>
        <div class="flex gap-4 mt-2">
          <Input
            :value="sourceTypeText"
            readonly
            placeholder="输入文字后自动检测语种"
            class="w-[200px]"
          />
          <div class="text-sm dark:text-slate-500">{{ sourceTip }}</div>
        </div>
        <Textarea
          v-model="sourceText"
          class="mt-4 h-[calc(100vh-190px)]"
          @input="inputSourceText"
        />
      </CardContent>
    </Card>
    <Card class="w-1/2">
      <CardContent>
        <LangSelector
          v-model="targetType"
          :list="langList"
          class="mt-4"
          @change="changeTargetLang"
        />
        <div class="mt-4 border border-slate-200 rounded-md p-2 h-[calc(100vh-200px)] overflow-auto">
          {{ targetText }}
        </div>
      </CardContent>
    </Card>
  </div>
</template>
<script setup lang="ts">
import {
  Card,
  CardContent,
} from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import LangSelector from '@/components/LangSelector.vue'
import { ref, computed } from 'vue'
import {
  detect,
  translate,
  updateTranslator,
} from '@rejax/browser-ai-translator' // @ts-ignore

// 语言标签名称
const langTagNames = ref(new Intl.DisplayNames(['zh'], { type: 'language' }))
const langList = computed(() => supportLangs.map(item => ({ value: item, label: langTagNames.value.of(item) })))
// 目前 API 支持的语种
const supportLangs = ['es', 'en', 'ja', 'zh', 'ar', 'bn', 'de', 'fr', 'hi', 'it', 'ko', 'nl', 'pl', 'pt', 'ru', 'th', 'tr', 'vi', 'zh-Hant']

const sourceType = ref('en')
const sourceTypeText = computed(() => {
  if (!sourceType.value) return ''
  return langTagNames.value.of(sourceType.value)
})
const sourceText = ref('')
const sourceTip = ref('')
let timer: NodeJS.Timeout | null = null
async function inputSourceText() {
  if (timer) clearTimeout(timer)
  timer = setTimeout(async () => {
    // 检测用户输入的文本语种
    await detectSourceLang()
    if (sourceType.value === targetType.value) {
      targetText.value = sourceText.value
      return
    }
    // 翻译
    await doTranslate()
    clearTimeout(timer)
    timer = null
  }, 1000)
}

const targetText = ref('')
const targetType = ref('es')
async function changeTargetLang() {
  if (!sourceText.value || targetType.value === sourceType.value) {
    targetText.value = sourceText.value
    return
  }
  const pair = {
    sourceLanguage: sourceType.value,
    targetLanguage: targetType.value,
  }
  // 更新 Translator
  await updateTranslator(pair)
  // 翻译
  await doTranslate()
}

// 翻译
async function doTranslate() {
  targetText.value = await translate({
    text: sourceText.value,
    sourceLanguage: sourceType.value,
    targetLanguage: targetType.value,
  })
}

// 检测用户输入的文本语种
async function detectSourceLang() {
  if (!sourceText.value) {
    sourceTip.value = ''
    sourceType.value = ''
    return
  }
  const res = await detect(sourceText.value)
  const { value } = res
  sourceType.value = value
  if (!supportLangs.includes(value)) {
    sourceTip.value = '暂不支持该语种'
    targetText.value = ''
  } else {
    sourceTip.value = ''
  }
}

</script>
