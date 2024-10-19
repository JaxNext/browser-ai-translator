<script setup lang="ts">
import { Button } from '@/components/ui/button'
import {
  Command,
  CommandEmpty,
  CommandGroup,
  CommandInput,
  CommandItem,
  CommandList,
} from '@/components/ui/command'

import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover'
import { cn } from '@/lib/utils'
import { Check, ChevronsUpDown } from 'lucide-vue-next'
import { ref } from 'vue'

const props = defineProps<{
  list: {
    value: string
    label: string
  }[]
}>()

const open = ref(false)
const value = defineModel<string>({
  required: true,
})
</script>

<template>
  <Popover v-model:open="open">
    <PopoverTrigger as-child>
      <Button
        variant="outline"
        role="combobox"
        :aria-expanded="open"
        v-bind="$attrs"
        class="w-[200px] justify-between"
      >
        {{ value
          ? props.list.find((item) => item.value === value)?.label
          : "请选择" }}
        <ChevronsUpDown class="ml-2 h-4 w-4 shrink-0 opacity-50" />
      </Button>
    </PopoverTrigger>
    <PopoverContent class="w-[200px] p-0">
      <Command>
        <CommandInput class="h-9" placeholder="搜索" />
        <CommandEmpty>无结果</CommandEmpty>
        <CommandList>
          <CommandGroup>
            <CommandItem
              v-for="item in props.list"
              :key="item.value"
              :value="item.value"
              @select="(ev) => {
                if (typeof ev.detail.value === 'string') {
                  value = ev.detail.value
                }
                open = false
                $emit('change', value)
              }"
            >
              {{ item.label }}
              <Check
                :class="cn(
                  'ml-auto h-4 w-4',
                  value === item.value ? 'opacity-100' : 'opacity-0',
                )"
              />
            </CommandItem>
          </CommandGroup>
        </CommandList>
      </Command>
    </PopoverContent>
  </Popover>
</template>