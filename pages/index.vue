<template>
  <section class="bg-[#041d3c] pb-10 lg:pb-12">
    <IndexHero />
    <div class="py-10 lg:py-12 gap-y-10 lg:gap-y-12 flex flex-col items-center">
      <Features />
      <GetStarted :is-next-step="false" />
    </div>
    <Buy />
    <Press />
  </section>
</template>
<script lang="ts" setup>
const { t } = useI18n();

const description = computed(() => t("head.descriptions.index"));

useSeoMeta({
  title: "",
  description,
  ogTitle: "",
  ogDescription: description,
});
</script>
