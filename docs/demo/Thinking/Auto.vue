<template>
  <div>
    <el-radio-group v-model="statusValue" style="margin-bottom: 12px;">
      <el-radio-button value="thinking">
        thinking
      </el-radio-button>
      <el-radio-button value="end">
        end
      </el-radio-button>
    </el-radio-group>
    <Thinking v-model="thinking" :status="statusValue" auto-collapse content="欢迎使用 Vue3_Chat" button-width="150px"
      max-width="100%" />
  </div>
</template>

<script setup lang="ts">
import { Thinking } from 'vue-chat-pro'
import type { ThinkingStatus } from 'vue-chat-pro/types'
const statusValue = ref<ThinkingStatus>('thinking')
import { ref } from 'vue'
const thinking = ref<boolean>(false)
</script>

<style scoped></style>