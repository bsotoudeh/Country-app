<template>
    <UInput
        v-model="searchValue"
        leading-icon="i-lucide-search"
        placeholder="Search for a country..."
        class="border-gray-100 w-full sm:w-1/3"
        :ui="{
            leading: 'h-8',
            leadingIcon: 'p-0 w-4 h-4',
        }"
    />
</template>

<script setup lang="ts">
interface Props {
  modelValue?: string
}

interface Emits {
  'update:modelValue': [value: string]
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

const searchValue = computed({
  get: () => props.modelValue || '',
  set: (value: string) => emit('update:modelValue', value)
})
</script>