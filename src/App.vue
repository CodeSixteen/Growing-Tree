<script lang="ts" setup>
const width = $ref(500)
const height = $ref(500)
const canvasRef = $ref<HTMLCanvasElement>()
const ctx = $computed(() => canvasRef!.getContext('2d')!)
let pendingTasks: Function[] = []

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  angle: number
}

let framesCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    framesCount += 1
    if (framesCount % 3 === 0)
      frame()
    startFrame()
  })
}
startFrame()

function init() {
  ctx.strokeStyle = 'red'
  step({
    start: { x: width / 2, y: height },
    length: 20,
    angle: -Math.PI / 2,
  })
}

function step(b: Branch, depth = 0) {
  const end = getEndPoint(b)
  drawLine(b)
  if (depth < 7 || Math.random() < 0.4) {
    pendingTasks.push(() => {
      step({
        start: end,
        length: b.length,
        angle: b.angle + Math.random() * 0.5,
      }, depth + 1)
    })
  }
  if (depth < 7 || Math.random() < 0.4) {
    pendingTasks.push(() => {
      step({
        start: end,
        length: b.length,
        angle: b.angle - Math.random() * 0.5,
      }, depth + 1)
    })
  }
}
function frame() {
  const tasks: Function[] = []
  pendingTasks = pendingTasks.filter((i) => {
    if (Math.random() > 0.4) {
      tasks.push(i)
      return false
    }
    return true
  })
  tasks.forEach(fn => fn())
}

function drawLine(b: Branch) {
  const end = getEndPoint(b)
  line(b.start, end)
}

function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.angle),
    y: b.start.y + b.length * Math.sin(b.angle),
  }
}

function line(a: Point, b: Point) {
  ctx.beginPath()
  ctx.moveTo(a.x, a.y)
  ctx.lineTo(b.x, b.y)
  ctx.closePath()
  ctx.stroke()
}

onMounted(() => {
  init()
})
</script>

<template>
  <canvas ref="canvasRef" class="mx-auto" border :width="width" :height="height" />
</template>
