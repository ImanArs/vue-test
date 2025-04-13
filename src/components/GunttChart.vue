<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import trucks from '../mock/truckRoutes.json'

const pxPerHour = 128
const hours = Array.from({ length: 24 }, (_, i) => i + 1)
const now = new Date()

const currentTimeOffset = computed(() => {
  const start = new Date(now)
  start.setHours(0, 0, 0, 0) // обнуляем до начала дня

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
  const pxPerHour = 128

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
    top: '0px',
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

  redLineX.value = Math.max(0, Math.min(x, rect.width)) // ограничим в пределах доски
}

onMounted(() => {
  window.addEventListener('mousemove', onDrag)
  window.addEventListener('mouseup', stopDrag)
})
</script>

<template>
  <div ref="boardRef" class="relative" :style="{ width: `${hours.length * pxPerHour + 128}px` }">
    <div class="text-black bg-white h-screen w-full overflow-auto">
      <div class="border-t border-black relative" :style="{ width: `${hours.length * pxPerHour + 128}px` }">
        <div class="border-b bg-white flex top-0 sticky z-10">
          <div class="shrink-0 w-32" />
          <div class="flex flex-1">
            <div
              v-for="hour in hours"
              :key="hour"
              class="text-xs text-black py-2 text-center border-l w-[128px] select-none"
            >
              {{ hour }}:00
            </div>
          </div>
        </div>

        <div v-for="truck in trucks" :key="truck.truck.id" class="border-b flex h-16">
          <div class="text-xs text-black p-2 bg-gray-100 shrink-0 w-32 select-none">
            {{ truck.truck.plate_number.trim() }}
          </div>

          <div class="flex-1 w-full select-none relative">
            <div
              v-for="route in truck.routes"
              :key="`route-${route.id}`"
            >
              <div
                v-for="job in route.jobs"
                :key="`job-${job.id}`"
                class="text-xs px-2 rounded-lg flex h-10 items-center absolute"
                :style="getJobStyle(job)"
                :class="getStatusColor(job.status)"
              >
                {{ job.pickup_location.name }}
              </div>
            </div>
          </div>
        </div>

        <div
          class="bg-red-600 w-[2px] cursor-ew-resize absolute z-20"
          :style="{ left: `${redLineX}px`, height: `${trucks.length * 128}px`, top: '0px' }"
          @mousedown="startDrag"
        />
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
