<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import { Branch, Point } from "./type";

const el = ref<HTMLCanvasElement>();
const ctx = computed(() => el.value!.getContext('2d')!)

const WIDTH = 800;
const HEIGHT = 600;

const pendingTasks: Function[] = []

function init() {
  ctx.value.strokeStyle = 'rgba(55, 65, 81, .3)'
  const branch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 40,
    theta: -Math.PI / 2
  }
  step(branch)
}

function step(b: Branch, depth = 0) {
  const end = getEndPoint(b)
  drawBranch(b)
  if (depth < 2 || Math.random() < 0.5) {
    pendingTasks.push(() =>
      step({
        start: end,
        length: b.length + Math.random() * 10 - 5,
        theta: b.theta - 0.3 * Math.random()
      }, depth + 1)
    )
  }
  if (depth < 2 || Math.random() < 0.5) {
    pendingTasks.push(() =>
      step({
        start: end,
        length: b.length + Math.random() * 10 - 5,
        theta: b.theta + 0.3 * Math.random()
      }, depth + 1)
    )
  }
}

function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

let frameCount = 0

function startFrame() {
  requestAnimationFrame(() => {
    frameCount += 1
    if (frameCount % 3 === 0)
      frame()
    startFrame()
  })
}

startFrame()

function getEndPoint(b: Branch) {
  const { start, length, theta } = b;
  return {
    x: start.x + length * Math.cos(theta),
    y: start.y + length * Math.sin(theta),
  }
}

function lineTo(p1: Point, p2: Point) {
  // start a new path
  ctx.value.beginPath()
  // move the pen to(30, 50)
  ctx.value.moveTo(p1.x, p1.y)
  // draw a line to(150, 100)
  ctx.value.lineTo(p2.x, p2.y)

  ctx.value.arc(p2.x, p2.y, 1, 0, Math.PI * 2);
  ctx.value.fillStyle = '#442525'
  ctx.value.fill()

  if (Math.random() < 0.5) {
    const img = new Image()
    img.src = './public/peach-flower.svg'
    ctx.value.drawImage(img, p2.x - img.width / 2, p2.y - img.height / 2)
  }
  if (Math.random() < 0.3) {
    const img = new Image()
    img.src = './public/peach-leave.svg'
    img.className = "test"
    ctx.value.drawImage(img, p2.x - img.width / 3, p2.y - img.height / 1)
  }
  ctx.value.stroke()
}

function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}

onMounted(() => {
  init()
})

</script>

<template>
  <canvas ref="el" width="800" height="600" />
</template>
