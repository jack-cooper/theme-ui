<script setup lang="ts">
import ThemeUi from "./theme-ui.vue";
</script>

<template>
  <div
    class="fixed bg-gradient-to-tr from-amber-500 via-red-700 to-85% to-gray-900 h-screen w-screen"
  />

  <div class="relative z-10">
    <Suspense>
      <ThemeUi class="py-8" />
    </Suspense>
  </div>
</template>

<style lang="postcss">
body {
  @apply text-zinc-200;
}
</style>
