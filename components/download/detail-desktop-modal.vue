<script lang="ts" setup>
const { t, localeProperties } = useI18n();

defineProps({
  version: {
    type: String,
    required: true,
  },
  date: {
    type: Number,
    required: true,
  },
  linkWalletFile: {
    type: String,
    required: true,
  },
  linkWalletCliFile: {
    type: String,
    required: true,
  },
  linkNodeFile: {
    type: String,
    required: true,
  },
  checksumWallet: {
    type: String,
    required: true,
  },
  checksumWalletCli: {
    type: String,
    required: true,
  },
  checksumNode: {
    type: String,
    required: true,
  },
});
</script>
<template>
  <section>
    <div>
      <h6 class="text-base font-bold">
        {{ $t("downloads.detailswindow.latestversion") }}
      </h6>
      <p>
        {{ version }} ({{
          convertUNIXTimestampToLocaleDate(date, localeProperties.iso)
        }}) &middot;
        <span class="text-beam-blue font-bold">
          <DownloadAlternativeLink
            :link="linkWalletFile"
            :title="t('downloads.type.wallet')"
          />
        </span>
        &middot;
        <span class="text-beam-blue font-bold">
          <DownloadAlternativeLink
            :link="linkWalletCliFile"
            :title="t('downloads.type.cli')"
          />
        </span>
        &middot;
        <span class="text-beam-blue font-bold">
          <DownloadAlternativeLink
            :link="linkNodeFile"
            :title="t('downloads.type.node')"
          />
        </span>
      </p>
    </div>
    <div v-if="checksumWallet" class="pt-3">
      <h6 class="font-bold">
        {{ $t("downloads.detailswindow.checksums.wallet") }}
      </h6>
      <LayoutCopyToClipboard :text="checksumWallet">
        {{ checksumWallet }}
      </LayoutCopyToClipboard>
    </div>
    <div v-if="checksumWalletCli" class="pt-3">
      <h6 class="font-bold">
        {{ $t("downloads.detailswindow.checksums.cli") }}
      </h6>
      <LayoutCopyToClipboard :text="checksumWalletCli">
        {{ checksumWalletCli }}
      </LayoutCopyToClipboard>
    </div>
    <div v-if="checksumNode" class="pt-3">
      <h6 class="font-bold">
        {{ $t("downloads.detailswindow.checksums.node") }}
      </h6>
      <LayoutCopyToClipboard :text="checksumNode">
        {{ checksumNode }}
      </LayoutCopyToClipboard>
    </div>
  </section>
</template>
