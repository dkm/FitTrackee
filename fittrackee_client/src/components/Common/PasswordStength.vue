<template>
  <div class="password-strength">
    <input
      class="password-slider"
      :class="`strength-${passwordScore}`"
      :style="{ backgroundSize: backgroundSize }"
      type="range"
      :value="passwordScore"
      min="0"
      max="4"
      step="1"
    />
    <div v-if="passwordStrength" class="password-strength-details">
      <span class="password-strength-value">
        {{ $t('user.PASSWORD_STRENGTH.LABEL') }}:
        {{ $t(`user.PASSWORD_STRENGTH.${passwordStrength}`) }}
      </span>
      <div class="info-box" v-if="passwordSuggestions.length > 0">
        <ul class="password-feedback">
          <li v-for="suggestion in passwordSuggestions" :key="suggestion">
            {{ $t(`user.PASSWORD_STRENGTH.SUGGESTIONS.${suggestion}`) }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { zxcvbn } from '@zxcvbn-ts/core'
  import {
    ComputedRef,
    Ref,
    computed,
    ref,
    onBeforeMount,
    toRefs,
    watch,
  } from 'vue'

  import { AUTH_USER_STORE, ROOT_STORE } from '@/store/constants'
  import { useStore } from '@/use/useStore'
  import { getPasswordStrength, setZxcvbnOptions } from '@/utils/password'

  interface Props {
    password: string
  }
  const props = defineProps<Props>()
  const { password } = toRefs(props)

  const store = useStore()
  const language: ComputedRef<string> = computed(
    () => store.getters[ROOT_STORE.GETTERS.LANGUAGE]
  )
  const isSuccess: ComputedRef<boolean> = computed(
    () => store.getters[AUTH_USER_STORE.GETTERS.IS_SUCCESS]
  )
  const passwordScore: Ref<number> = ref(0)
  const passwordStrength: Ref<string> = ref('')
  const passwordSuggestions: Ref<string[]> = ref([])
  const backgroundSize = ref('0% 100%')

  onBeforeMount(async () => await setZxcvbnOptions(language.value))

  function calculatePasswordStrength(password: string) {
    const zxcvbnResult = zxcvbn(password)
    passwordScore.value = zxcvbnResult.score
    passwordStrength.value = getPasswordStrength(passwordScore.value)
    passwordSuggestions.value = zxcvbnResult.feedback.suggestions
    backgroundSize.value = (passwordScore.value * 100) / 4 + '% 100%'
  }

  watch(
    () => language.value,
    async (newLanguageValue) => {
      await setZxcvbnOptions(newLanguageValue)
    }
  )
  watch(
    () => password.value,
    async (newPassword) => {
      if (isSuccess.value) {
        passwordStrength.value = ''
      } else {
        calculatePasswordStrength(newPassword)
      }
    }
  )
</script>

<style lang="scss" scoped>
  @import '~@/scss/vars.scss';

  .password-strength {
    cursor: default;
    display: flex;
    flex-direction: column;

    @mixin slider-background-image($color) {
      background: var(--password-bg-color);
      background-image: -webkit-gradient(
        linear,
        20% 0%,
        20% 100%,
        color-stop(0%, $color),
        color-stop(100%, $color)
      );
      background-image: -webkit-linear-gradient(left, $color 0%, $color 100%);
      background-image: -moz-linear-gradient(left, $color 0%, $color 100%);
      background-image: -o-linear-gradient(to right, $color 0%, $color 100%);
      background-image: linear-gradient(to right, $color 0%, $color 100%);
      background-repeat: no-repeat;
    }
    .password-slider {
      -webkit-appearance: none;
      appearance: none;
      border: none;
      border-radius: 8px;
      height: 5px;
      outline: none;
      padding: 0;
    }
    .strength-0,
    .strength-1 {
      @include slider-background-image(var(--password-color-weak));
    }
    .strength-2 {
      @include slider-background-image(var(--password-color-medium));
    }
    .strength-3 {
      @include slider-background-image(var(--password-color-good));
    }
    .strength-4 {
      @include slider-background-image(var(--password-color-strong));
    }

    .password-slider::-webkit-slider-thumb,
    .password-slider::-moz-range-thumb {
      opacity: 0;
    }
    .password-slider::-webkit-slider-thumb {
      -webkit-appearance: none;
    }
    .password-slider::-moz-range-thumb {
      appearance: none;
    }

    .password-strength-details {
      margin-bottom: $default-margin * 0.5;
      margin-top: -1 * $default-margin;
      padding: 0 $default-padding;

      .password-strength-value {
        font-size: 0.85em;
      }
      .info-box {
        padding: $default-padding * 0.1 $default-padding;
        .password-feedback {
          padding-left: $default-padding * 2;
        }
      }
    }
  }
</style>
