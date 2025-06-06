<template>
  <div ref="divRef" :class="[ns.b(), ns.b(placement)]">
    <div v-if="avatar || slots.avatar" :class="[ns.b('avatar'), props.className?.avatar]" :style="props.styles?.avatar">
      <slot name="avatar">
        <el-avatar :size="32" :src="avatar" v-if="typeof avatar === 'string'" />
        <component :is="avatar" v-else />
      </slot>
    </div>
    <div v-if="slots.header || slots.footer" :class="[ns.b('content-wrapper')]">
      <div v-if="slots.header" :class="[ns.b('header'), props.className?.header]" :style="props.styles?.header">
        <slot name="header"></slot>
      </div>
      <div :class="[
        ns.b('content'),
        ns.b(`content-${props.variant}`),
        props.className?.content,
        props.shape && ns.b(`content-${props.placement}-${props.shape}`),
        {
          [ns.b('content-filled-transparent')]: props.transparent
        }
      ]" :style="props.styles?.content">
        <template v-if="props.loading">
          <slot v-if="slots.loading" name="loading" />
          <component :is="loadingRender()" v-else-if="loadingRender" />
          <Loading v-else />
        </template>
        <template v-if="!props.loading">
          <template v-if="slots.content">
            <slot v-if="slots.content" name="content"></slot>
          </template>
          <template v-else>
            <!-- 根据 mergedContent 类型选择渲染方式 -->
            <component :is="mergeContent" v-if="!isString(mergeContent)" />
            <div v-else v-html="mergeContent"></div>
          </template>
        </template>
      </div>
      <div v-if="slots.footer" :class="[ns.b('footer'), props.className?.footer]" :style="props.styles?.footer">
        <slot name="footer"></slot>
      </div>
    </div>
    <template v-else>
      <div :class="[
        ns.b('content'),
        ns.b(`content-${props.variant}`),
        props.className?.content,
        props.shape && ns.b(`content-${props.placement}-${props.shape}`),
        {
          [ns.b('content-filled-transparent')]: props.transparent
        }
      ]" :style="props.styles?.content">
        <template v-if="props.loading">
          <slot v-if="slots.loading" name="loading" />
          <component :is="loadingRender()" v-else-if="loadingRender" />
          <Loading v-else />
        </template>
        <template v-if="!props.loading">
          <template v-if="slots.content">
            <slot v-if="slots.content" name="content"></slot>
          </template>
          <template v-else>
            <!-- 根据 mergedContent 类型选择渲染方式 -->
            <component :is="mergeContent" v-if="!isString(mergeContent)" />
            <div v-else v-html="mergeContent"></div>
          </template>
        </template>
      </div>
      <div v-if="slots.footer" :class="[ns.b('footer'), props.className?.footer]" :style="props.styles?.footer">
        <slot name="footer"></slot>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import type { BubbleProps } from './types'
import type { Ref } from 'vue'
import { computed, useSlots, watch, ref } from 'vue'
import useTypingConfig from './hooks/useTypingConfig'
import useTypedEffect2 from './hooks/useTypedEffect2'

import { useClassMoudle } from '@/hooks/useClassMoudle'
import Loading from './components/Loading.vue'

defineOptions({
  inheritAttrs: false
})

const props = withDefaults(defineProps<BubbleProps>(), {
  placement: 'start',
  loading: false,
  variant: 'filled',
  loadingRender: undefined,
  content: '',
  avatar: '',
  typing: false,
  transparent: false
})
console.log('bubble props', props)

const emit = defineEmits(['onTypingComplete', 'onUpdate'])

const ns = useClassMoudle('bubble')
// footer content avatar header loading

const slots = useSlots()
console.log('bubble slots', slots)

const divRef = ref<HTMLDivElement>()

const contents = computed(() => props.content)

const [typingEnabled, interval, step] = useTypingConfig(props.typing)

const [typedContent, isTyping] = useTypedEffect2(
  contents as Ref<string>,
  typingEnabled as boolean,
  step as number,
  interval as number
)

console.log('typedContent.value', typedContent.value)

const mergeContent = computed(() =>
  props.messageRender
    ? props.messageRender((typedContent.value as () => string)?.() ?? '')
    : typedContent.value
)


function isString(content: unknown) {
  return typeof content === 'string'
}


const triggerTypingCompleteRef = ref(false)
watch(
  () => isTyping.value || props.loading,
  () => {
    if (!isTyping.value && !props.loading) {
      if (!triggerTypingCompleteRef.value) {
        triggerTypingCompleteRef.value = true
        emit('onTypingComplete')
      }
    } else {
      triggerTypingCompleteRef.value = false
    }
  }
)


// 内容更新触发的回调
watch(
  () => typedContent.value(),
  () => emit('onUpdate')
)


defineExpose({
  nativeElement: divRef
})
</script>

<style scoped lang="scss">
@import './style';
</style>
