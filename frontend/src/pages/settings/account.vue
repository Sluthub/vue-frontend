<template>
  <settings-page page-title="settings.account.account">
    <template #actions>
      <VBtn
        v-if="auth.currentUser"
        color="primary"
        class="ml-a"
        @click="onUploadClick"
      >
        {{ $t('settings.account.updateImage') }}
      </VBtn>
      <input
        ref="uploader"
        type="file"
        name="file"
        accept="image/*"
        style="display: none"
        @change="select($event.target.files)"
      />
      <VBtn
        v-if="auth.currentUser"
        color="error"
        class="ml-a"
        @click="deleteUserImage"
      >
        {{ $t('settings.account.removeImage') }}
      </VBtn>
    </template>
    <template #content>
      <VCol class="text-center" cols="10" md="10" lg="3">
        <user-image
          v-if="auth.currentUser"
          :user="auth.currentUser"
          :size="200"
          :key="componentKey"
          rounded
          class="text-center"
        />
          <!-- :initialLoading="initialLoading" -->
        <h1 class="mt-3 text-center" v-if="auth.currentUser">
          {{ auth.currentUser.Name }}
        </h1>
      </VCol>
      <VCol cols="10" md="10" lg="7">
        <VCard class="pa-0 flex-grow-1">
          <VForm v-model="valid">
            <VCardTitle>{{
              $t('settings.account.updatePassword')
            }}</VCardTitle>
            <VCardText class="pb-0">
              <VTextField
                class="px-5 pt-5"
                v-model="currentPassword"
                variant="outlined"
                :label="$t('settings.account.currentPassword')"
                :append-icon="showPassword ? IconEyeOff : IconEye"
                :type="showPassword ? 'text' : 'password'"
                @click:append="() => (showPassword = !showPassword)"
              ></VTextField>
              <VTextField
                class="px-5"
                v-model="newPassword"
                variant="outlined"
                :label="$t('settings.account.newPassword')"
                :append-icon="showPassword2 ? IconEyeOff : IconEye"
                :type="showPassword2 ? 'text' : 'password'"
                @click:append="() => (showPassword2 = !showPassword2)"
              ></VTextField>
              <VTextField
                class="px-5"
                v-model="confirmPassword"
                variant="outlined"
                :label="$t('wizard.confirmPassword')"
                :append-icon="showPassword2 ? IconEyeOff : IconEye"
                :type="showPassword2 ? 'text' : 'password'"
                :rules="SamePasswordRules"
                @click:append="() => (showPassword2 = !showPassword2)"
              ></VTextField>
            </VCardText>
            <VCardActions
              class="d-flex align-center pt-0"
              :class="{
                'justify-end pr-9': !$vuetify.display.mobile,
                'justify-center': $vuetify.display.mobile
              }"
            >
              <VBtn
                color="primary"
                variant="depressed"
                :disabled="!valid"
                width="12em"
                @click="updatePassword"
              >
                {{ $t('settings.account.updatePassword') }}
              </VBtn>
            </VCardActions>
          </VForm>
        </VCard>
      </VCol>
    </template>
  </settings-page>
</template>

<script setup lang="ts">
import { shallowRef, ref } from 'vue';
import { useI18n } from 'vue-i18n';
import { useRemote, useSnackbar } from '@/composables';
import { getImageApi } from '@jellyfin/sdk/lib/utils/api/image-api';
import { getUserApi } from '@jellyfin/sdk/lib/utils/api/user-api';
import IconEyeOff from 'virtual:icons/mdi/eye-off';
import IconEye from 'virtual:icons/mdi/eye';

const { t } = useI18n();
const remote = useRemote();
const auth = useRemote().auth;

const valid = ref(false);
const componentKey = ref(0);
const currentPassword = ref('');
const newPassword = ref('');
const confirmPassword = ref('');
const showPassword = ref(false);
const showPassword2 = ref(false);

const SamePasswordRules = [
  (v: string): boolean | string =>
    v === newPassword.value || t('validation.bothPasswordsSame')
];

function clearFields(): void {
  currentPassword.value = '';
  newPassword.value = '';
  confirmPassword.value = '';
}

async function updatePassword(): Promise<void> {
  try {
    await remote.sdk.newUserApi(getUserApi).updateUserPassword({
      userId: remote.auth.currentUserId ?? '',
      updateUserPassword: {
        CurrentPw: currentPassword.value,
        NewPw: newPassword.value
      }
    });
    useSnackbar(
      t('settings.account.passwordChangedSuccessfully'),
      'success'
    );
    clearFields();
  } catch (error) {
    // eslint-disable-next-line no-console
    console.error(error);
    useSnackbar(
      t('settings.account.passwordChangeError'),
      'error'
    );
  }
}

async function deleteUserImage(): Promise<void> {
  try {
    await remote.sdk.newUserApi(getImageApi).deleteUserImage({
      userId: remote.auth.currentUserId ?? '',
      imageType: 'Primary'
    });

    useSnackbar(
      t('settings.account.deleteImageSuccess'),
      'success'
    );
    reload();
  } catch (error) {
    useSnackbar(
      t('settings.account.deleteImageError'),
      'error'
    );
    // eslint-disable-next-line no-console
    console.error(error);
  }
}

async function postUserImage(file: string): Promise<void> {
  try {
    await remote.sdk.newUserApi(getImageApi).postUserImage({
      userId: remote.auth.currentUserId ?? '',
      imageType: 'Primary',
      body: file
    });

    useSnackbar(
      t('settings.account.updateImageSuccess'),
      'success'
    );
    reload();
  } catch (error) {
    useSnackbar(
      t('settings.account.updateImageError'),
      'error'
    );
    // eslint-disable-next-line no-console
    console.error(error);
  }
}

const uploader = shallowRef<HTMLInputElement>();

function onUploadClick(): void {
  if(uploader.value) {
    uploader.value.click();
  }
}

function select(files: FileList): void {
  const reader = new FileReader();
  reader.onload = (e: ProgressEvent): void => {
    // Taken from jellyfin-apiclient-javascript method 'uploadUserImage'
    // Split by a comma to remove the url: prefix
    const data = ((e.target as FileReader).result as string).split(',')[1];
    postUserImage(data);
  };
  reader.readAsDataURL(files[0]);
}

function reload() {
  //This updates the component key which is passed through to the user-image component
  componentKey.value += 1;
}

// export default Vue.extend({
//   data() {
//     return {
//     };
//   },
//   computed: {
//     ...mapStores(authStore, snackbarStore)
// }}
// );
</script>
