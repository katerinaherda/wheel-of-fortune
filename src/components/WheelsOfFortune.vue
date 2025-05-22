<template>
  <div id="wheelOfFortune">
    <canvas ref="canvasRef" id="wheel" width="300" height="300"></canvas>
    <div ref="spinRef" id="spin">SPIN</div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'

// Typ pro sektor
type Sector = {
  color: string
  label: string
}

const sectors: Sector[] = [
  { color: '#f82', label: 'Stack' },
  { color: '#0bf', label: '10' },
  { color: '#fb0', label: '200' },
  { color: '#0fb', label: '50' },
  { color: '#b0f', label: '100' },
  { color: '#f0b', label: '5' },
  { color: '#bf0', label: '500' },
  { color: 'black', label: 'test' },
  { color: 'blue', label: 'test' },
  { color: 'green', label: 'test' },
  { color: 'pink', label: 'test' },
  { color: 'brown', label: 'test' },
]

const canvasRef = ref<HTMLCanvasElement | null>(null)
const spinRef = ref<HTMLDivElement | null>(null)

onMounted(() => {
  const canvas = canvasRef.value
  const spinEl = spinRef.value

  if (!canvas || !spinEl) {
    throw new Error('Elementy canvas nebo spin nebyly nalezeny')
  }

  const ctx = canvas.getContext('2d')
  if (!ctx) {
    throw new Error('Nepodařilo se získat 2D context')
  }

  const rand = (m: number, M: number): number => Math.random() * (M - m) + m
  const tot = sectors.length
  const dia = canvas.width
  const rad = dia / 2
  const PI = Math.PI
  const TAU = 2 * PI
  const arc = TAU / tot

  const friction = 0.991
  let angVel = 0
  let ang = 0

  const getIndex = (): number => Math.floor(tot - (ang / TAU) * tot) % tot

  function drawSector(sector: Sector, i: number): void {
    const ang = arc * i
    ctx.save()
    ctx.beginPath()
    ctx.fillStyle = sector.color
    ctx.moveTo(rad, rad)
    ctx.arc(rad, rad, rad, ang, ang + arc)
    ctx.lineTo(rad, rad)
    ctx.fill()

    ctx.translate(rad, rad)
    ctx.rotate(ang + arc / 2)
    ctx.textAlign = 'right'
    ctx.fillStyle = '#fff'
    ctx.font = 'bold 30px sans-serif'
    ctx.fillText(sector.label, rad - 10, 10)
    ctx.restore()
  }

  function rotate(): void {
    const sector = sectors[getIndex()]
    canvas.style.transform = `rotate(${ang - PI / 2}rad)`
    spinEl.textContent = !angVel ? 'SPIN' : sector.label
    spinEl.style.background = sector.color
  }

  function frame(): void {
    if (!angVel) return
    angVel *= friction
    if (angVel < 0.002) angVel = 0
    ang += angVel
    ang %= TAU
    rotate()
  }

  function engine(): void {
    frame()
    requestAnimationFrame(engine)
  }

  function init(): void {
    sectors.forEach(drawSector)
    rotate()
    engine()
    spinEl.addEventListener('click', () => {
      if (!angVel) angVel = rand(0.25, 0.45)
    })
  }

  init()
})
</script>
