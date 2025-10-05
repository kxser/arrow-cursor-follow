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

const cols = 20
const rows = 20
const total = cols * rows

const mousePosition = ref({ x: 0, y: 0 })
const gridEl = ref(null)
const gridOffset = ref({ left: 0, top: 0 })

let animationFrameId = null
const throttledMousePosition = ref({ x: 0, y: 0 })

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

function calculateAngle(arrowCenterX, arrowCenterY, mouseX, mouseY) {
  const dx = mouseX - (gridOffset.value.left + arrowCenterX)
  const dy = mouseY - (gridOffset.value.top + arrowCenterY)

  return Math.atan2(dy, dx) * (180 / Math.PI)
}

const arrows = computed(() => {
  const mouseX = throttledMousePosition.value.x
  const mouseY = throttledMousePosition.value.y



  return arrowPositions.map(arrow => ({
    ...arrow,
    angle: calculateAngle(arrow.centerX, arrow.centerY, mouseX, mouseY)
  }))
})

async function updateGridOffset() {
  await nextTick()
  const rect = gridEl.value?.getBoundingClientRect()
  if (rect) {

    gridOffset.value = {
      left: rect.left,
      top: rect.top,
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
