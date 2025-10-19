<template>
  <div class="min-h-screen">
    <header
      :class="colorMode.value === 'dark' ? 'bg-gray-900' : 'bg-white'"
      class="sticky top-0 z-50 w-full h-16 flex justify-between items-center px-8 2xl:px-40 mx-auto">
      <h1 class="text-lg font-semibold">Where in the world?</h1>
      <UColorModeButton />
    </header>
    <slot />
  </div>
</template>

<script setup lang="ts">
const colorMode = useColorMode();
</script>
