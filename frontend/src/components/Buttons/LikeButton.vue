<template>
  <VBtn
    :icon="isFavorite ? IMdiHeart : IMdiHeartOutline"
    :size="size"
    :color="isFavorite ? 'primary' : undefined"
    :loading="loading"
    @click.stop.prevent="isFavorite = !isFavorite" />
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { BaseItemDto } from '@jellyfin/sdk/lib/generated-client';
import { getUserLibraryApi } from '@jellyfin/sdk/lib/utils/api/user-library-api';
import IMdiHeart from 'virtual:icons/mdi/heart';
import IMdiHeartOutline from 'virtual:icons/mdi/heart-outline';
import { useRemote } from '@/composables';

const props = withDefaults(
  defineProps<{ item: BaseItemDto; size?: string }>(),
  {
    size: 'small'
  }
);
const remote = useRemote();
const loading = ref(false);
const isFavoriteOverride = ref<boolean | undefined>(undefined);

const isFavorite = computed({
  get() {
    return isFavoriteOverride.value === undefined
      ? props.item.UserData?.IsFavorite ?? false
      : isFavoriteOverride.value;
  },
  async set(newValue) {
    try {
      if (!props.item.Id) {
        throw new Error('Item has no Id');
      }

      loading.value = true;

      await (newValue
        ? remote.sdk.newUserApi(getUserLibraryApi).markFavoriteItem({
          userId: remote.auth.currentUserId ?? '',
          itemId: props.item.Id
        })
        : remote.sdk.newUserApi(getUserLibraryApi).unmarkFavoriteItem({
          userId: remote.auth.currentUserId ?? '',
          itemId: props.item.Id
        }));

      isFavoriteOverride.value = newValue;
    } catch {} finally {
      loading.value = false;
    }
  }
});
</script>
