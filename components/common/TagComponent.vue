<template>
  <div
    class="px-3 text-sm transition-all duration-300 rounded-full w-fit"
    :class="{
      'bg-secondaryLight text-secondary': color === 'info',
      'border border-emerald-200 bg-emerald-50 text-emerald-600': color === 'emerald',
      'border border-rose-200 bg-rose-50 text-rose-600': color === 'rose',
      'border border-amber-200 bg-amber-50 text-amber-600': color === 'amber',
      'border border-zinc-200 bg-zinc-50 text-zinc-600': color === 'zinc',
      'border border-indigo-200 bg-indigo-50 text-indigo-600': color === 'indigo',
      'border border-purple-200 bg-purple-50 text-purple-600': color === 'purple',
      'hover:cursor-pointer hover:shadow': tag.link,
    }"
    @click="goTo"
  >
    {{ tag.name }}
  </div>
</template>

<script setup>
  const props = defineProps({
    tag: {
      type: Object,
      default: () => {},
    },
    color: {
      type: String,
      default: 'info',
    },
  })

  const goTo = async () => {
    await navigateTo({ path: props.tag.link })
  }
</script>
