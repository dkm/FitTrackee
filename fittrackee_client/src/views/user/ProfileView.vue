<template>
  <div id="profile" class="view" v-if="authUser.username">
    <router-view :user="authUser"></router-view>
    <div id="bottom" />
  </div>
</template>

<script setup lang="ts">
  import { ComputedRef, computed } from 'vue'

  import { AUTH_USER_STORE } from '@/store/constants'
  import { IAuthUserProfile } from '@/types/user'
  import { useStore } from '@/use/useStore'

  const store = useStore()

  const authUser: ComputedRef<IAuthUserProfile> = computed(
    () => store.getters[AUTH_USER_STORE.GETTERS.AUTH_USER_PROFILE]
  )
</script>

<style lang="scss" scoped>
  @import '~@/scss/vars.scss';
  #profile {
    display: flex;
    flex-direction: column;
    padding: 0 $default-padding;
    margin-bottom: 40px;

    ::v-deep(.profile-form) {
      display: flex;
      flex-direction: column;

      hr {
        border-color: var(--card-border-color);
        border-width: 1px 0 0 0;
      }

      .form-items {
        display: flex;
        flex-direction: column;

        input {
          margin: $default-padding * 0.5 0;
        }

        select {
          height: 35px;
          padding: $default-padding * 0.5 0;
        }
        ::v-deep(.custom-textarea) {
          textarea {
            padding: $default-padding * 0.5;
          }
        }

        .form-item {
          display: flex;
          flex-direction: column;
          padding: $default-padding;
        }
        .birth-date {
          height: 20px;
        }
      }

      .form-buttons {
        display: flex;
        margin-top: $default-margin;
        padding: $default-padding 0;
        gap: $default-padding;
      }
    }
  }
</style>
