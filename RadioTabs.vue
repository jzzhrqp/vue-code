<template>
  <div class="switch-box-parent">
    <div class="switch-box" ref="switchBoxRef">
      <div class="highlight" :style="highlightStyle"></div>
      <div
        v-for="(option, index) in options"
        :key="option.value"
        class="option"
        @click="handleClick(option, index)"
        :ref="(el) => (optionRefs[index] = el)"
      >
        <span :class="{ 'selected-text': option.selected }">{{ option.label }}</span>

        <v-icon v-if="option.value === active" class="ml-16px" size="20" color="#28B884">
          mdi-check-circle
        </v-icon>
        <v-icon v-else class="ml-16px" size="20" color="#C6C8CD"> mdi-check-circle </v-icon>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, defineProps, defineEmits, onMounted, watch } from 'vue'
const switchBoxRef = ref<HTMLElement | null>(null)
const switchBoxWidth = ref(0)
const optionRefs = ref([] as any[])
const optionWidths = ref([] as any[])
interface Option {
  label: string
  value: string | number
  selected: boolean
}

const props = defineProps<{
  options: Option[]
  modelValue: string | number
  active: string | number
}>()

const emits = defineEmits(['update:modelValue'])

const selectedIndex = ref(props.options.findIndex((opt) => opt.selected))
// if (selectedIndex.value === -1) {
//   selectedIndex.value = 0
// }

const highlightStyle = computed(() => {
  // 计算 highlight 的 left 位置
  let left = optionWidths.value.slice(0, selectedIndex.value).reduce((acc, width) => acc + width, 0)

  // 对于最左边的 option，增加 8px 左边距
  if (selectedIndex.value === 0) {
    left += 8
  } else {
    left += 8 * selectedIndex.value
  }

  // 计算 highlight 的宽度
  const highlightWidth = optionWidths.value[selectedIndex.value]
  console.log(
    'left',
    left,
    'highlightWidth',
    highlightWidth,
    'selectedIndex.value',
    selectedIndex.value,
    'optionWidths.value',
    optionWidths.value
  )
  return {
    left: `${(left / switchBoxWidth.value) * 100}%`,
    width: `${(highlightWidth / switchBoxWidth.value) * 100}%`
  }
})

const handleClick = (option: Option, index: number) => {
  props.options.forEach((opt) => {
    opt.selected = false
    if (opt.value && option.value && opt.value === option.value) {
      opt.selected = true
    }
  })
  selectedIndex.value = index
  emits('update:modelValue', option.value)
}

watch(
  () => props.active,
  (val) => {
    console.log('watch active', val)
    if (val) {
      if (!props.options || props.options.length === 0) return
      props.options.forEach((opt) => {
        opt.selected = false
        if (opt.value && val && opt.value === val) {
          opt.selected = true
        }
      })
      selectedIndex.value = props.options.findIndex((opt) => opt.selected)
      if (selectedIndex.value === -1) {
        selectedIndex.value = 0
      }
    }
  },
  { immediate: true }
)
onMounted(() => {
  // 获取 switchBox 和每个 option 的宽度
  setTimeout(() => {
    if (switchBoxRef.value) {
      switchBoxWidth.value = switchBoxRef.value.offsetWidth
      optionWidths.value = optionRefs.value?.map((el) => el.offsetWidth)
      console.log(optionWidths.value)
    }
  }, 100)
})
</script>

<style scoped>
.switch-box-parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
.switch-box {
  position: relative;
  width: auto;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background-color: #f0f1f2;
  border-radius: 30px;
  box-sizing: content-box;
}

.highlight {
  position: absolute;
  top: 6px;
  height: 40px;
  bottom: 6px;
  background: #ffffff;
  border: 1px solid #7ed4b5;
  border-radius: 30px;
  opacity: 1;
  box-shadow: 0px 4px 5px 0px rgba(9, 15, 37, 0.06);
  transition: left 0.3s ease-in-out;
}

.option {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  position: relative;
  z-index: 1;
  display: flex;
  height: 40px;
  padding-right: 16px;
  padding-left: 16px;
  margin: 6px 8px 6px 0;
  font-family: Arial-Bold, Arial, serif;
  font-size: 14px;
  font-weight: bold;
  line-height: 20px;
  color: #c6c8cd;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
}

.checkmark-icon:before {
  display: block;
  text-align: center;
  background-color: grey;
  border-radius: 50%;
  content: '✔';
}
.selected-text {
  font-size: 14px;
  font-family: Arial-Bold, Arial, serif;
  font-weight: bold;
  color: #363c47;
  line-height: 20px;
  transition: all 0.3s ease-in-out;
}
</style>
