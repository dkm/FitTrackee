<template>
  <div class="dropdown-wrapper">
    <div class="dropdown-selected" @click="toggleDropdown">
      <slot></slot>
    </div>
    <ul class="dropdown-list" v-if="isOpen">
      <li
        class="dropdown-item"
        :class="{ selected: option.value === selected }"
        v-for="(option, index) in dropdownOptions"
        :key="index"
        @click="updateSelected(option)"
      >
        {{ option.label }}
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
  import { ref, watch } from 'vue'
  import { useRoute } from 'vue-router'

  import { IDropdownOption, TDropdownOptions } from '@/types/forms'
  interface Props {
    options: TDropdownOptions
    selected: string
  }
  const props = defineProps<Props>()

  const emit = defineEmits({
    selected: (option: IDropdownOption) => option,
  })

  const route = useRoute()
  const isOpen = ref(false)
  const dropdownOptions = props.options.map((option) => option)

  function toggleDropdown() {
    isOpen.value = !isOpen.value
  }
  function updateSelected(option: IDropdownOption) {
    emit('selected', option)
    isOpen.value = false
  }

  watch(
    () => route.path,
    () => (isOpen.value = false)
  )
</script>

<style scoped lang="scss">
  .dropdown-list {
    list-style-type: none;
    background-color: #ffffff;
    padding: 0 !important;
    margin-top: 5px;
    margin-left: -20px !important;
    position: absolute;
    text-align: left;
    border: solid 1px lightgrey;
    box-shadow: 2px 2px 5px lightgrey;
    width: auto !important;

    li {
      padding: 3px 8px;
    }
  }

  .dropdown-item {
    cursor: pointer;

    &.selected {
      font-weight: bold;
    }

    &.selected::after {
      content: ' ✔';
    }

    &:hover {
      background-color: var(--dropdown-hover-color);
    }
  }
</style>
