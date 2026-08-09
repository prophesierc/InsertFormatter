<script setup lang="ts">
withDefaults(
  defineProps<{
    type?: "button";
  }>(),
  {
    type: "button",
  },
);
</script>

<template>
  <button
    class="rounded bg-slate-700 px-3 py-1.5 text-sm text-slate-200 hover:bg-slate-600"
  >
    <slot />
  </button>
</template>
