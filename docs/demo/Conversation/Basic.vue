<template>
  <Conversation v-model:active-key="activeKey" :items="items" style="width: 300px" />
</template>

<script setup lang="ts">
import { Conversation } from 'vue-chat-pro'
import { ref } from 'vue'

const items = Array.from({ length: 4 }).map((_, index) => ({
  key: `item${index + 1}`,
  label: `Conversation Item ${index + 1}`,
  disabled: index === 3,
}))
const activeKey = ref('item1')
</script>

<style scoped></style>