<script setup lang="ts">
import parser from 'cron-parser'
import dayjs from 'dayjs'
import { ref, watch } from 'vue'

const toast = useToast()

const input = useTemplateRef('input')

const help = [
  { keyword: '*', desc: '匹配任意值' },
  { keyword: ',', desc: '匹配多个值' },
  { keyword: '-', desc: '匹配指定范围' },
  { keyword: '/', desc: '匹配指定间隔' },
]

const cronExpr = ref<string>('5 4 * * *')
const nextTimes = ref<Date[]>([])
const moreTimes = ref(false)

watch(cronExpr, () => parseCrontab(), { immediate: true })

function parseCrontab() {
  if (!cronExpr.value || cronExpr.value.length < 5) {
    nextTimes.value = []
    return
  }

  try {
    const interval = parser.parse(cronExpr.value)

    const results = []
    for (let i = 0; i < 5; i++) {
      results.push(interval.next().toDate())
    }
    nextTimes.value = results
  }
  catch (e) {
    console.error(e)
    nextTimes.value = []
  }
}

function handleCopy() {
  navigator.clipboard.writeText(cronExpr.value).then(() => {
    toast.add({
      title: '已复制到剪贴板',
      color: 'warning',
      duration: 2000,
    })
  }).catch(() => {
    toast.add({
      title: '复制失败，请手动复制',
      color: 'error',
      duration: 2000,
    })
  })
}

function handleSelect(type: 'minute' | 'hour' | 'day' | 'month' | 'week') {
  const map = ['minute', 'hour', 'day', 'month', 'week']
  const arr = cronExpr.value.trim().split(' ')
  const index = map.indexOf(type)

  if (arr.length !== 5 || index === -1)
    return

  const start = arr.slice(0, index).reduce((acc, val) => acc + val.length + 1, 0)
  const end = start + arr[index].length

  input.value?.focus()
  input.value?.setSelectionRange(start, end)
}
</script>

<template>
  <UApp>
    <div class="min-h-screen bg-[#171717] text-[#FAFAFA] text-center">
      <div class="flex flex-col items-center max-w-[1100px] mx-auto">
        <div class="my-20">
          <div class="text-5xl text-[#FFFF80] font-bold">
            Crontab Guru
          </div>
          <div class="mt-6">
            Cron 计划表达式的快速简单编辑器
          </div>
        </div>

        <div class="mb-8 space-y-2">
          <div v-for="(time, index) in (moreTimes ? nextTimes : nextTimes.slice(0, 1))" :key="index" class="flex items-center">
            <div v-if="index === 0" class="font-bold italic text-2xl flex items-baseline space-x-2 mb-4 text-gray-400">
              <div>「</div>
              <div class="mr-4 underline cursor-pointer" @click="moreTimes = !moreTimes">
                下次
              </div>
              <div class="text-[#FAFAFA]">
                {{ dayjs(time).format('YYYY-MM-DD HH:mm') }}
              </div>
              <div>执行</div>
              <div>」</div>
            </div>
            <div v-else class="flex w-full justify-center text-gray-400 space-x-2">
              <div>之后</div>
              <div class="text-[#FAFAFA]">
                {{ dayjs(time).format('YYYY-MM-DD HH:mm') }}
              </div>
              <div>执行</div>
            </div>
          </div>
        </div>

        <div class="w-3/4 flex items-center">
          <input ref="input" v-model="cronExpr" class="w-full outline-0 text-center border-3 border-[#FFFF80] h-18 text-3xl rounded-md shadow-[0_0_10px_#ffff80aa]" placeholder="* * * * *">
          <button class="bg-[#ffff80] text-[#171717] font-bold rounded-md py-0.5 -ml-20 w-16 hover:bg-[#ffff80e8] cursor-pointer" @click="handleCopy">
            复制
          </button>
        </div>

        <div class="grid grid-cols-5 gap-8 mt-6">
          <div class="text-gray-400 cursor-pointer underline" @click="handleSelect('minute')">
            分钟
          </div>
          <div class="text-gray-400 cursor-pointer underline" @click="handleSelect('hour')">
            小时
          </div>
          <div class="text-gray-400 cursor-pointer underline" @click="handleSelect('day')">
            日期
          </div>
          <div class="text-gray-400 cursor-pointer underline" @click="handleSelect('month')">
            月份
          </div>
          <div class="text-gray-400 cursor-pointer underline" @click="handleSelect('week')">
            星期
          </div>
        </div>

        <div class="mt-10 w-2/4">
          <div v-for="item, index in help" :key="index" class="flex items-baseline text-gray-400 border-b border-b-gray-500 py-2">
            <div class="w-2/5 text-end mr-8">
              {{ item.keyword }}
            </div>
            <div class="flex-1 text-start">
              {{ item.desc }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </UApp>
</template>

<style scoped>
input::selection {
  background-color: rgba(255, 255, 128, 0.2);
  color: #ffff80;
}
input {
  box-shadow: 0 0 10px #ffff80aa;
}
input:focus {
  box-shadow: 0 0 20px #ffff80aa;
}
</style>
