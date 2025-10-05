<template>
  <div
    class="icon-grid"
    ref="gridEl"
    :style="{
      display: 'inline-grid',
      width: 'fit-content',
      gridTemplateColumns: `repeat(${cols}, ${iconSize}px)`,
      background: '#000',
      border: '4px solid #444',
      borderRadius: '12px'
    }"
  >
    <UIcon
      v-for="arrow in arrows"
      :key="arrow.id"
      name="i-heroicons-arrow-long-right-16-solid"
      :size="iconSize"
      class="arrow"
      :style="{
        transform: `rotate(${arrow.angle}deg)`,
        color: '#fff'
      }"
    />
  </div>
</template>

<script setup>
const baseSize = 16
const scale = 2  
const iconSize = baseSize * scale
const activationRadius = 150
const defaultAngle = 0

const cols = 20
const rows = 20
const total = cols * rows

const mousePosition = ref({ x: 0, y: 0 })
const gridEl = ref(null)
const gridOffset = ref({ left: 0, top: 0, width: 0, height: 0 })

let animationFrameId = null
const throttledMousePosition = ref({ x: 0, y: 0 })

function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max)
}

function distanceFromGrid(mouseX, mouseY) {
  const { left, top, width, height } = gridOffset.value
  if (!width || !height) return Infinity
  const clampedX = clamp(mouseX, left, left + width)
  const clampedY = clamp(mouseY, top, top + height)
  return Math.hypot(mouseX - clampedX, mouseY - clampedY)
}

function updateMouse(e) {
  mousePosition.value.x = e.clientX
  mousePosition.value.y = e.clientY
  if (!animationFrameId) {
    animationFrameId = requestAnimationFrame(() => {
      throttledMousePosition.value = { ...mousePosition.value }

      animationFrameId = null
    })
  }
}

const arrowPositions = Array.from({ length: total }, (_, idx) => {
  const col = idx % cols
  const row = Math.floor(idx / cols)
  return {
    id: idx,
    x: col * iconSize,
    y: row * iconSize,
    centerX: col * iconSize + iconSize / 2,
    centerY: row * iconSize + iconSize / 2,
  }
})

const currentAngles = ref(arrowPositions.map(() => defaultAngle))
let angleAnimationId = null

function calculateAngle(arrowCenterX, arrowCenterY, mouseX, mouseY) {
  const dx = mouseX - (gridOffset.value.left + arrowCenterX)
  const dy = mouseY - (gridOffset.value.top + arrowCenterY)

  return Math.atan2(dy, dx) * (180 / Math.PI)
}

function shortestAngleDifference(target, current) {
  let diff = (target - current + 540) % 360 - 180
  return diff
}

const targetAngles = computed(() => {
  const mouseX = throttledMousePosition.value.x
  const mouseY = throttledMousePosition.value.y
  const isActive = distanceFromGrid(mouseX, mouseY) <= activationRadius

  return arrowPositions.map(arrow =>
    isActive
      ? calculateAngle(arrow.centerX, arrow.centerY, mouseX, mouseY)
      : defaultAngle
  )
})

const arrows = computed(() =>
  arrowPositions.map((arrow, idx) => ({
    ...arrow,
    angle: currentAngles.value[idx] ?? defaultAngle
  }))
)

function animateAngles() {
  const targets = targetAngles.value
  const nextAngles = currentAngles.value.slice()
  let hasChanges = false

  for (let i = 0; i < nextAngles.length; i++) {
    const target = targets[i]
    const current = nextAngles[i]
    const diff = shortestAngleDifference(target, current)

    if (Math.abs(diff) > 0.1) {
      nextAngles[i] = current + diff * 0.18
      hasChanges = true
    } else if (current !== target) {
      nextAngles[i] = target
      hasChanges = true
    }
  }

  if (hasChanges) {
    currentAngles.value = nextAngles
  }

  angleAnimationId = requestAnimationFrame(animateAngles)
}

async function updateGridOffset() {
  await nextTick()
  const rect = gridEl.value?.getBoundingClientRect()
  if (rect) {

    gridOffset.value = {
      left: rect.left,
      top: rect.top,
      width: rect.width,
      height: rect.height,
    }
  }
}

let resizeTimeout = null
function throttledUpdateGridOffset() {
  if (resizeTimeout) return
  resizeTimeout = setTimeout(() => {
    updateGridOffset()
    resizeTimeout = null
  }, 100)
}

onMounted(() => {
  updateGridOffset()
  angleAnimationId = requestAnimationFrame(animateAngles)
  window.addEventListener('resize', throttledUpdateGridOffset, { passive: true })
  window.addEventListener('scroll', throttledUpdateGridOffset, { passive: true })
  window.addEventListener('mousemove', updateMouse, { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('resize', throttledUpdateGridOffset)

  window.removeEventListener('scroll', throttledUpdateGridOffset)

  
  window.removeEventListener('mousemove', updateMouse)
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
  if (resizeTimeout) clearTimeout(resizeTimeout)
  if (angleAnimationId) {
    cancelAnimationFrame(angleAnimationId)
    angleAnimationId = null
  }
})
</script>

<style scoped>
.icon-grid {
  display: grid;
  gap: 0;
  line-height: 0;
  transform: translateZ(0);
  will-change: transform;

}

.arrow {
  transform-origin: center;
  will-change: transform;
}
</style>
