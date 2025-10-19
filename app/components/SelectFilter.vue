<template>
  <USelect
    placeholder="Filter by Region"
    v-model="selectedValue"
    :items="options"
    class="w-48"
  />
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  modelValue?: string
}>()

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()

const selectedValue = computed({
  get: () => props.modelValue || '',
  set: (value: string) => emit('update:modelValue', value)
})

const options = [
  { label: 'All', value: 'all' },
  { label: 'Europe', value: 'europe' },
  { label: 'Asia', value: 'asia' },
  { label: 'Africa', value: 'africa' },
  { label: 'Americas', value: 'americas' },
  { label: 'Oceania', value: 'oceania' }
]
</script>