<template>
  <VContainer
    class="fill-height"
    fluid>
    <VRow justify="center">
      <VCol
        sm="6"
        md="6"
        lg="5">
        <h1 class="text-h4 mb-6 text-center">
          Please wait while we establish a secure connection to the server...
        </h1>
        <!-- <AddServerForm /> -->
      </VCol>
    </VRow>
  </VContainer>
</template>

<route lang="yaml">
meta:
  layout: server
</route>

<script setup lang="ts">
import { useRoute } from 'vue-router';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
const route = useRoute();

route.meta.title = "Connecting...";
</script>
