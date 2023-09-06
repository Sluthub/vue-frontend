<template>
  <div v-if="children">
    <VTabs
      v-model="currentTab"
      class="mb-3"
      bg-color="transparent">
      <VTab
        v-for="(baseItems, type) in children"
        :key="type"
        :value="type">
        {{ type }} ({{ baseItems.length }})
      </VTab>
    </VTabs>
    <h1
      v-if="!children && !loading"
      class="text-h5 text-center">
      {{ $t('collectionEmpty') }}
    </h1>
    <VWindow
      v-model="currentTab"
      class="bg-transparent">
      <VWindowItem
        v-for="(baseItems, type) in children"
        :key="type"
        :value="type">
        <VContainer>
          <SkeletonItemGrid
            v-if="loading"
            :view-type="item.Type" />
          <ItemGrid
            :loading="loading"
            :items="baseItems" />
        </VContainer>
      </VWindowItem>
    </VWindow>
  </div>
</template>

<script setup lang="ts">
import { BaseItemDto } from '@jellyfin/sdk/lib/generated-client';
import { groupBy } from 'lodash-es';
import { computed, onMounted, ref, watch } from 'vue';
import { itemsStore } from '@/store';

const props = defineProps<{
  item: BaseItemDto;
}>();

const items = itemsStore();

const currentTab = ref(0);
const loading = ref(false);
const children = computed(() => {
  return groupBy(items.getChildrenOfParent(props.item.Id), 'Type');
});

async function refreshCollection(itemId: string | undefined) {
  loading.value = true;

  try {
    /**
     * Only try to load children if there aren't any children items in the itemStore yet
     */
    await items.fetchAndAddCollection(itemId);
  } catch (error) {
    console.error(error);
    useSnackbar(t('failedToRefreshItems'), 'error');
  } finally {
    loading.value = false;
  }
}

onMounted(() => refreshCollection((props.item as { Id: string }).Id));

watch(
  () => props.item,
  async (item) => {
    await refreshCollection(item.Id);
  },
  { immediate: true }
);
</script>
