<template>
  <h1 class="text-3xl text-center font-bold dark:text-slate-50">浏览器内置 AI 翻译</h1>
  <div class="mt-4 flex gap-4 h-[calc(100vh-100px)]">
    <Card class="w-1/2">
      <CardContent>
        <Input
          v-model="sourceTypeText"
          readonly
          placeholder="输入文字后自动检测语种"
          class="w-[200px] mt-4"
        />
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
import { ref, computed, onMounted } from 'vue'
import {
  detect,
  translate,
  setOriginTrialToken,
  updateTranslator,
  checkTranslatorAvailability,
} from '@rejax/browser-ai'
import { useToast } from '@/components/ui/toast/use-toast'
import {
  DETECT_ORIGIN_TRIAL_TOKEN,
  TRANSLATOR_TRIAL_TOKEN,
} from '@/constants/trial'

const { toast } = useToast()

// 语言标签名称
const langTagNames = ref(new Intl.DisplayNames(['zh'], { type: 'language' }))
const langList = computed(() => supportLangs.map(item => ({ value: item, label: langTagNames.value.of(item) })))
// 目前 API 支持的语种
const supportLangs = ['es', 'en', 'ja', 'zh', 'ar', 'bn', 'de', 'fr', 'hi', 'it', 'ko', 'nl', 'pl', 'pt', 'ru', 'th', 'tr', 'vi', 'zh-Hant']

const sourceType = ref('')
const sourceTypeText = computed(() => {
  if (!sourceType.value) return ''
  return langTagNames.value.of(sourceType.value)
})
const sourceText = ref('')
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
const targetType = ref('zh')
const targetTypeText = computed(() => {
  if (!targetType.value) return ''
  return langTagNames.value.of(targetType.value)
})
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
  try {
    await updateTranslator(pair)
    // 翻译
    await doTranslate()
  } catch (error) {
    console.error('111', error)
    showToast(error.message)
  }
}

// 翻译
async function doTranslate() {
  try {
    targetText.value = await translate({
      text: sourceText.value,
    sourceLanguage: sourceType.value,
      targetLanguage: targetType.value,
    })
  } catch (error) {
    console.error('222', error)
    showToast(error.message)
  }
}

// 检测用户输入的文本语种
async function detectSourceLang() {
  if (!sourceText.value) {
    sourceType.value = ''
    return
  }
  const res = await detect(sourceText.value)
  const { value } = res
  sourceType.value = value
  if (!supportLangs.includes(value)) {
    showToast('暂不支持该语种')
    targetText.value = ''
  }
}

function showToast(msg: string) {
  toast({
    description: `当前语种不支持「${sourceTypeText.value}」→「${targetTypeText.value}」`,
    variant: 'destructive',
  })
}

onMounted(() => {
  setOriginTrialToken(DETECT_ORIGIN_TRIAL_TOKEN)
  setOriginTrialToken(TRANSLATOR_TRIAL_TOKEN)
})
</script>
