<script setup lang="ts">
import { Expand } from 'lucide-vue-next'
import { computed, onMounted, ref } from 'vue'

import trucks from '../mock/truckRoutes.json'

const pxPerHour = 64
const hours = Array.from({ length: 24 }, (_, i) => i + 1)
const now = new Date()

const currentTimeOffset = computed(() => {
  const start = new Date(now)
  start.setHours(0, 0, 0, 0)

  const diffMs = now.getTime() - start.getTime()
  const minutes = diffMs / 1000 / 60

  return (minutes / 60) * pxPerHour
})

function getStatusColor(status: string) {
  if (status.includes('FINISHED'))
    return 'bg-gray-500'
  if (status.includes('IN_PROGRESS'))
    return 'bg-blue-600'
  return 'bg-blue-300'
}

function getJobStyle(job: any) {
  const start = new Date(job.plan_start_at)
  const end = new Date(job.plan_end_at)

  const startHours = start.getUTCHours() + start.getUTCMinutes() / 60 + start.getUTCSeconds() / 3600
  const endHours = end.getUTCHours() + end.getUTCMinutes() / 60 + end.getUTCSeconds() / 3600

  let duration = endHours - startHours
  if (duration < 0)
    duration += 24

  return {
    left: `${startHours * pxPerHour - pxPerHour}px`,
    width: `calc(${duration * pxPerHour}px)`,
  }
}

const redLineX = ref(currentTimeOffset.value)
const boardRef = ref<HTMLElement | null>(null)
let isDragging = false

function startDrag() {
  isDragging = true
}

function stopDrag() {
  isDragging = false
}

function onDrag(event: MouseEvent) {
  if (!isDragging || !boardRef.value)
    return

  const rect = boardRef.value.getBoundingClientRect()
  const x = event.clientX - rect.left

  redLineX.value = Math.max(0, Math.min(x, rect.width))
}

onMounted(() => {
  window.addEventListener('mousemove', onDrag)
  window.addEventListener('mouseup', stopDrag)
})
</script>

<template>
  <div ref="boardRef" class="relative">
    <div class="mx-auto border-2 border-[#d3d3d3] rounded-[10px_10px_0px_0px] bg-white w-[1028px] relative overflow-hidden">
      <div class="text-black p-2 border-b border-[#d3d3d3] flex items-center left-0 top-0 justify-between">
        <h2 class="text-[20px] font-semibold">
          Route Schedule
        </h2>
        <button>
          <Expand />
        </button>
      </div>

      <div class="text-black mx-auto p-10 h-auto min-h-[338px] w-[1028px] relative overflow-auto">
        <div class="relative" :style="{ width: `${hours.length * pxPerHour + 64}px` }">
          <div class="bg-white flex top-0 sticky z-10">
            <div class="shrink-0 w-20" />
            <div class="flex flex-1">
              <div
                v-for="hour in hours"
                :key="hour"
                class="text-xs text-black py-2 text-center flex flex-col gap-2 w-[64px] select-none items-center"
              >
                <span>{{ hour }}:00</span>
                <div class="bg-black h-2 w-[1px]" />
              </div>
            </div>
          </div>

          <div v-for="truck in trucks" :key="truck.truck.id" class="flex h-16 items-center">
            <div class="text-base text-white p-2 rounded-[6px_0px_0px_6px] bg-[#333] shrink-0 max-h-10 w-20 select-none">
              {{ truck.truck.plate_number.trim() }}
            </div>

            <div class="flex h-[60px] h-full w-full select-none items-center relative">
              <div
                v-for="route in truck.routes"
                :key="`route-${route.id}`"
                class="bg-red-500 h-10 relative"
              >
                <div
                  v-for="job in route.jobs"
                  :key="`job-${job.id}`"
                  class="text-xs px-2 rounded-[12px] flex h-10 items-center absolute"
                  :style="getJobStyle(job)"
                  :class="getStatusColor(job.status)"
                >
                  {{ job.pickup_location.name }}
                </div>
              </div>
            </div>
          </div>

          <div
            class="bg-black w-[2px] cursor-ew-resize top-[32px] absolute z-20"
            :style="{ left: `${redLineX}px`, height: `${trucks.length * 100}px` }"
            @mousedown="startDrag"
          >
            <div class="rounded-full bg-black h-4 w-4 transform top-0 absolute -translate-x-[50%] -left-4" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
body {
  background-color: aliceblue;
  font-family: sans-serif;
}
</style>
