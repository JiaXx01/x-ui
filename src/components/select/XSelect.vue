<template>
  <Teleport to="body">
    <div
      id="x-selector"
      class="absolute left-1/2 -translate-x-1/2 top-10 border rounded bg-white p-2 pb-0 outline-none text-gray-500"
      tabindex="-1"
      v-if="status"
      @blur="onSelectorLoseFocus"
    >
      <input
        id="x-select-input"
        class="border rounded outline-none px-1 h-6 text-sm"
        :placeholder="placeholder ? placeholder : ''"
        style="width: 500px"
        ref="inputRef"
        v-model="inputContent"
        @blur="onInputLoseFocus"
        v-focus
      />
      <ul
        class="mt-2 max-h-72 overflow-auto x-scrollbar"
        ref="optionsListRef"
        @mouseleave="curMouseIndex = null"
      >
        <li
          class="rounded px-1 cursor-pointer last:mb-2 flex items-center"
          :class="{
            'bg-gray-50': index === curMouseIndex,
            '!bg-blue-700 text-white': index === curKeyboardIndex
          }"
          v-for="(item, index) in _options"
          :key="index"
          @mouseenter="curMouseIndex = index"
          @click="clickItem(index)"
        >
          <span class="mr-1" v-if="item?.icon">
            <svg class="icon" aria-hidden="true">
              <use :xlink:href="item.icon"></use>
            </svg>
          </span>
          <span class="px-1" v-html="item.label"></span>
          <span class="ml-auto text-xs">{{ item.remarks }}</span>
        </li>
      </ul>
    </div>
  </Teleport>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

interface OptionProp {
  icon?: string
  label: string
  remarks?: string
  onSelect: () => void
  similar?: number
}

interface Props {
  modelValue: boolean
  options: OptionProp[]
  placeholder?: string
  width?: number | string
}

const props = withDefaults(defineProps<Props>(), {
  width: '500'
})
const emits = defineEmits(['update:modelValue'])

// 组件显示状态
const status = ref(false)
watch(
  () => props.modelValue,
  newValue => {
    status.value = newValue
  },
  { immediate: true }
)
watch(status, newStatus => {
  emits('update:modelValue', newStatus)
  setKeyboardEvent()

  reset()
})

const reset = () => {
  inputContent.value = ''
  curKeyboardIndex.value = 0
  curMouseIndex.value = null
}

/**
 * 组件选项
 */
const _options = ref(props.options)
const inputContent = ref('')

const filterOptions = () => {
  if (!inputContent.value) return (_options.value = props.options)
  const queryStr = inputContent.value.toLowerCase()
  const queryStrArr = queryStr.split('')

  // 元字符处理
  let regStr = '(.*?)'
  queryStrArr.forEach(str => {
    regStr += str.replace(/([.?*+^$[\]\\(){}|-])/g, '\\$1') + '(.*?)'
  })
  const reg = RegExp(regStr, 'i')

  const items: OptionProp[] = []

  props.options.forEach(_item => {
    const item = Object.assign({}, _item)
    const isMatch = reg.test(item.label)

    if (isMatch) {
      const queryStrArr = queryStr.split('')
      const keywordIndexArr: number[] = []
      // 计算关键字索引
      let keywordIndex = 0
      queryStrArr.forEach(keyword => {
        const index = item.label.indexOf(keyword, keywordIndex)
        keywordIndex = index + 1
        keywordIndexArr.push(index)
      })

      // 根据索引替换样式
      const labelStrArr = item.label.split('')
      keywordIndexArr.forEach(index => {
        if (index === -1) return
        const s = labelStrArr[index]
        labelStrArr.splice(
          index,
          1,
          `<span class="text-blue-200 font-bold">${s}</span>`
        )
      })
      item.label = labelStrArr.join('')

      // 计算相似度
      const lowerLabel = item.label.toLowerCase()
      const similar = getSimilar(lowerLabel, queryStr)
      item.similar = similar

      items.push(item)
    }
  })

  // 根据相似度排序
  items.sort((a, b) => {
    if (a.similar && b.similar) {
      return b.similar - a.similar
    } else return 0
  })

  _options.value = items
}

// 计算文本相似度算法
const getSimilar = (s: string, t: string): number => {
  if (!s || !t) {
    return 0
  }
  if (s === t) {
    return 100
  }
  var l = s.length > t.length ? s.length : t.length
  var n = s.length
  var m = t.length
  var d: number[][] = []
  var min = function (a: number, b: number, c: number) {
    return a < b ? (a < c ? a : c) : b < c ? b : c
  }
  var i, j, si, tj, cost
  if (n === 0) return m
  if (m === 0) return n
  for (i = 0; i <= n; i++) {
    d[i] = []
    d[i][0] = i
  }
  for (j = 0; j <= m; j++) {
    d[0][j] = j
  }
  for (i = 1; i <= n; i++) {
    si = s.charAt(i - 1)
    for (j = 1; j <= m; j++) {
      tj = t.charAt(j - 1)
      if (si === tj) {
        cost = 0
      } else {
        cost = 1
      }
      d[i][j] = min(d[i - 1][j] + 1, d[i][j - 1] + 1, d[i - 1][j - 1] + cost)
    }
  }
  let res: number = (1 - d[n][m] / l) * 100
  return res
}

watch(inputContent, () => {
  filterOptions()
  curKeyboardIndex.value = 0
})

/**
 *  组件交互
 */
// 自动聚焦
const vFocus = {
  mounted: (el: { focus: () => void }) => el.focus()
}

// 隐藏组件
const onSelectorLoseFocus = (e: FocusEvent) => {
  if (
    e.relatedTarget &&
    (e.relatedTarget as HTMLElement).id === 'x-select-input'
  )
    return
  status.value = false
}
const onInputLoseFocus = (e: FocusEvent) => {
  if (e.relatedTarget && (e.relatedTarget as HTMLElement).id === 'x-selector')
    return
  status.value = false
}

// 选项滚动
const curMouseIndex = ref<null | number>(null)
const curKeyboardIndex = ref(0)
const optionsListRef = ref<HTMLElement | null>(null)
const inputRef = ref<HTMLInputElement | null>(null)

const scrollToItem = () => {
  if (!optionsListRef.value || !inputRef.value) return
  const index = curKeyboardIndex.value
  const item = optionsListRef.value.children[index]

  const selectorScrollTop = optionsListRef.value.scrollTop

  const selectorClient = optionsListRef.value.getBoundingClientRect()
  const itemClient = item.getBoundingClientRect()

  const topDistance = itemClient.top - selectorClient.top
  if (topDistance < 0) {
    optionsListRef.value.scrollTop = selectorScrollTop + topDistance
  }
  const bottomDistance = itemClient.bottom - selectorClient.bottom
  if (bottomDistance > 0) {
    optionsListRef.value.scrollTop = selectorScrollTop + bottomDistance
  }
}

// 键盘事件
const onKeyboardDown = (e: KeyboardEvent) => {
  e.stopPropagation()
  const code = e.code
  const index: number = curKeyboardIndex.value

  switch (code) {
    case 'ArrowUp':
      if (index !== 0) {
        curKeyboardIndex.value--
      }
      break
    case 'ArrowDown':
      if (index !== _options.value.length - 1) {
        curKeyboardIndex.value++
      }
      break
    case 'Enter':
      _options.value[curKeyboardIndex.value].onSelect()
      status.value = false
      break
    case 'Escape':
      status.value = false
      break
    default:
      break
  }

  if (['ArrowDown', 'ArrowUp'].includes(code)) {
    scrollToItem()
  }
}

const setKeyboardEvent = () => {
  if (status.value) {
    window.addEventListener('keydown', onKeyboardDown)
  } else {
    window.removeEventListener('keydown', onKeyboardDown)
  }
}

const clickItem = (index: number) => {
  _options.value[index].onSelect()
  status.value = false
}
</script>

<style scoped>
.x-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.x-scrollbar::-webkit-scrollbar-track {
  border-radius: 2px;
}

.x-scrollbar::-webkit-scrollbar-thumb {
  --tw-bg-opacity: 1;
  background: rgb(243 244 246 / var(--tw-bg-opacity));
  border-radius: 10px;
}
.x-scrollbar::-webkit-scrollbar-thumb:hover {
  --tw-bg-opacity: 1;
  background-color: rgb(229 231 235 / var(--tw-bg-opacity));
}

.icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
