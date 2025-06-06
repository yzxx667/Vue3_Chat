<script setup lang="ts">
import type { Component } from 'vue'

import type { ListItemType } from './hooks/useListData'
import type { BubbleDataType, BubbleListProps, scrollTopParameters } from './types'

import { nextTick, onMounted, ref, watch } from 'vue'
import Bubble from '../Bubble/Bubble.vue'
import { useClassMoudle } from '@/hooks/useClassMoudle'
import useDisplayData from './hooks/useDisplayData'
import useListData from './hooks/useListData'

const props = withDefaults(defineProps<BubbleListProps>(), {
  autoScroll: true,
  items: () => [],
})

const slots = defineSlots<{
  avatar?: (slotProps: { info: BubbleDataType }) => void
  header?: (slotProps: { info: BubbleDataType }) => void
  loading?: (slotProps: { info: BubbleDataType }) => void
  footer?: (slotProps: { info: BubbleDataType }) => void
}>()

const ns = useClassMoudle('bubble-list')

const TOLERANCE = 1
const initialized = ref(true)
const scrollReachEnd = ref(false)
const updateCount = ref(0)
const listRef = ref<HTMLElement>()
const bubbleRefs = ref<Record<string, InstanceType<typeof Bubble>>>({})
const { ListData, setListData } = useListData(props.items, props.roles)
const [displayData, onTypingComplete, ItemsWatch] = useDisplayData(ListData)

watch(
  () => props.items,
  () => {
    setListData(props.items)
  }
)

watch(
  () => ListData.value,
  () => {
    ItemsWatch()
  }
)

function onInternalScroll(e: Event) {
  const target = e.target as HTMLElement
  // 兼容 .0625rem 以内的误差
  scrollReachEnd.value =
    target.scrollHeight - Math.abs(target.scrollTop) - target.clientHeight <= TOLERANCE
}

watch(
  () => updateCount.value,
  () => {
    if (props.autoScroll && listRef.value && scrollReachEnd.value) {
      listRef.value!.scrollTo({
        top: listRef.value!.scrollHeight,
      })
    }
  }
)

watch(
  () => displayData.value.length,
  () => {
    nextTick(() => {
      if (props.autoScroll) {
        updateCount.value += 1
        scrollReachEnd.value = true
      }
    })
    // nextTick(() => {
    //   if (props.autoScroll) {
    //     const lastItemKey = displayData.value[displayData.value.length - 2]?.key
    //     const bubbleInst = bubbleRefs.value[lastItemKey!]
    //     if (bubbleInst) {
    //       const { nativeElement } = bubbleInst
    //       const { top = 0, bottom = 0 } = nativeElement?.getBoundingClientRect() ?? {}
    //       const { top: listTop, bottom: listBottom } = listRef.value!.getBoundingClientRect()
    //       const isVisible = top < listBottom && bottom > listTop
    //       if (isVisible) {
    //         updateCount.value += 1
    //         scrollReachEnd.value = true
    //       }
    //     }
    //   }
    // })
  }
)

function onBubbleUpdate() {
  if (props.autoScroll) {
    updateCount.value = updateCount.value + 1
  }
}

function onTypingCompleteFn(bubble: ListItemType) {
  if (!bubble?.key) return
  bubble?.onTypingComplete?.()
  onTypingComplete(bubble.key)
}

function scrollTo({ key, offset, behavior = 'smooth', block }: scrollTopParameters) {
  console.log('scrollTo', { key, offset, behavior, block });

  if (typeof offset === 'number') {
    // Offset scroll

    listRef.value!.scrollTo({
      top: offset,
      behavior,
    })
  } else if (key !== undefined) {
    // 滚动到某个bubble
    // Key scroll
    const bubbleInst = bubbleRefs.value![key as number]

    if (bubbleInst) {
      // Block current auto scrolling
      const index = displayData.value.findIndex((dataItem) => dataItem.key === key)
      scrollReachEnd.value = index === displayData.value.length - 1

      // Do native scroll
      bubbleInst.$el.scrollIntoView({
        behavior,
        block,
      })
    }
  }
}

function getBubbleRefs(
  node: Component<InstanceType<typeof Bubble>> | null,
  key: number | string | undefined
) {
  if (key === null || key === undefined) return
  if (node) {
    Reflect.set(bubbleRefs.value, key, node)
  } else {
    Reflect.deleteProperty(bubbleRefs.value, key)
  }
}

onMounted(() => {
  nextTick(() => {
    scrollTo({ offset: listRef.value!.scrollHeight, behavior: 'auto' })
  })
})

defineExpose({
  nativeElement: listRef.value,
  scrollTo,
})
</script>

<template>
  <div ref="listRef" :class="[ns.b(), rootClassName, className, scrollReachEnd && ns.b('reach-end')]"
    @scroll="onInternalScroll">
    <Bubble v-for="bubble in displayData" :key="bubble.key" v-bind="bubble"
      :ref="(node) => getBubbleRefs(node, bubble.key)" :on-typing-complete="() => onTypingCompleteFn(bubble)"
      :on-update="onBubbleUpdate" :typing="initialized ? (bubble.typing as boolean) : false">
      <template v-for="(_slot, slotName) in slots" :key="slotName" #[slotName]="slotProps">
        <slot :name="slotName" :info="{ ...slotProps, ...bubble }" />
      </template>
    </Bubble>
  </div>
</template>

<style lang="scss">
// @import './instyledex';</style>
