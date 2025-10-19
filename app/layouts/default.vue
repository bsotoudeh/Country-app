<template>
  <div class="min-h-screen">
    <header
      :class="colorMode.value === 'dark' ? 'bg-gray-900' : 'bg-white'"
      class="sticky top-0 z-50 w-full h-16 flex justify-between items-center px-8 lg:px-32">
      <!-- <template #title> -->
        <h1 class="text-lg font-semibold">Where in the world?</h1>
      <!-- </template> -->
      <!-- <template #right> -->
        <UColorModeButton />
      <!-- </template> -->
    </header>
    <slot />
  </div>
</template>

<script setup lang="ts">
const colorMode = useColorMode();
</script>
