<script lang="ts" setup>
defineProps({
  title: {
    type: String,
    required: true,
  },
  image: {
    type: String,
    required: true,
  },
  link: {
    type: String,
    required: true,
  },
  linkText: {
    type: String,
    required: false,
    default: "",
  },
});
</script>
<template>
  <LayoutLink
    :to="link"
    class="p-4 group flex flex-col border rounded-xl border-black border-opacity-30 transition md:p-7 shadow-[0px_0px_0px_1px_rgba(255,255,255,.05)_inset] focus:outline-none focus:ring-2 focus:ring-offset-2 w-full h-full"
  >
    <Icon class="w-auto h-[32px]" :name="image" :as-image="true" />
    <h3
      class="mt-4 text-white/80 mb-3 text-lg font-bold group-hover:text-inherit transition-colors"
    >
      {{ title }}
    </h3>
    <div class="mt-auto gap-4">
      <LayoutExternalLink>{{
        linkText != "" ? linkText : $t("feature.learnmore")
      }}</LayoutExternalLink>
    </div>
  </LayoutLink>
</template>
