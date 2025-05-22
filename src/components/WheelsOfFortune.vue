<template>
  <div class="flex h-full justify-center items-center">
    <div id="wheelOfFortune">
      <canvas
        ref="canvasRef"
        id="wheel"
        width="600"
        height="600"
        class="w-[300px] h-[300px] lg:w-[600px] lg:h-[600px]"
      ></canvas>
      <div ref="spinRef" id="spin">SPIN</div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'

type Sector = {
  color: string
  label: string
  image?: string
}

const sectors: Sector[] = [
  { color: '#b0f', label: '100', image: '/images/cat1.png' },
  { color: '#f0b', label: '5', image: '/images/cat2.png' },
  { color: '#bf0', label: '500', image: '/images/deer.svg' },
  { color: '#b0f', label: '100', image: '/images/cat1.png' },
  { color: '#f0b', label: '5', image: '/images/cat2.png' },
  { color: '#bf0', label: '500', image: '/images/deer.svg' },
  { color: '#b0f', label: '100', image: '/images/cat1.png' },
  { color: '#f0b', label: '5', image: '/images/cat2.png' },
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

  const rand = (m: number, M: number) => Math.random() * (M - m) + m

  const tot = sectors.length
  const elSpin = document.querySelector('#spin')

  const dia = ctx.canvas.width

  const rad = dia / 2
  const PI = Math.PI
  const TAU = 2 * PI
  const arc = TAU / sectors.length
  const friction = 0.991 // 0.995=soft, 0.99=mid, 0.98=hard
  const angVelMin = 0.002 // Below that number will be treated as a stop
  let angVelMax = 0 // Random ang.vel. to acceletare to
  let angVel = 0 // Current angular velocity
  let ang = 0 // Angle rotation in radians
  let isSpinning = false
  let isAccelerating = false

  //* Get index of current sector */
  const getIndex = (): number => Math.floor(tot - (ang / TAU) * tot) % tot

  const loadImage = (
    ctx: CanvasRenderingContext2D,
    sector: { image: string },
    rad: number,
    rot: number,
  ) => {
    const img = new Image()
    img.onload = function () {
      ctx.save()
      // Posun na střed kola
      ctx.translate(rad, rad)
      // Natočení podle sektoru
      ctx.rotate(rot)
      // Posun na pozici, kde má být obrázek (např. vzdálenost od středu)
      const offset = 220
      ctx.translate(offset, 0)
      // Otočení obrázku o 90° doleva
      ctx.rotate(-Math.PI / -2)
      // Vykreslení obrázku se zarovnáním na střed
      const imgWidth = 145
      const imgHeight = 145
      ctx.drawImage(img, -imgWidth / 2, -imgHeight / 2, imgWidth, imgHeight)
      ctx.restore()
    }
    img.src = sector.image
  }

  //* Draw sectors and prizes texts to canvas */
  const drawSector = (sector, i) => {
    const ang = arc * i
    // COLOR
    ctx.beginPath()
    ctx.fillStyle = sector.color
    ctx.moveTo(rad, rad)
    ctx.arc(rad, rad, rad, ang, ang + arc)
    ctx.lineTo(rad, rad)
    ctx.fill()

    // TEXT
    const rot = ang + arc / 2
    ctx.save()
    ctx.translate(rad, rad)
    ctx.rotate(rot)
    ctx.textAlign = 'right'
    ctx.fillStyle = '#fff'
    ctx.font = 'bold 30px sans-serif'
    ctx.fillText(sector.label, rad - 10, 10)

    // IMG
    loadImage(ctx, sector, rad, rot)

    ctx.restore()
  }

  //* CSS rotate CANVAS Element */
  const rotate = () => {
    const sector = sectors[getIndex()]
    ctx.canvas.style.transform = `rotate(${ang - PI / 2}rad)`
    elSpin.textContent = !angVel ? 'SPIN' : sector.label
    elSpin.style.background = sector.color
  }

  const frame = () => {
    if (!isSpinning) return
    if (angVel >= angVelMax) isAccelerating = false
    // Accelerate
    if (isAccelerating) {
      angVel ||= angVelMin // Initial velocity kick
      angVel *= 1.06 // Accelerate
    }
    // Decelerate
    else {
      isAccelerating = false
      angVel *= friction // Decelerate by friction
      // SPIN END:
      if (angVel < angVelMin) {
        isSpinning = false
        angVel = 0
      }
    }

    ang += angVel // Update angle
    ang %= TAU // Normalize angle
    rotate() // CSS rotate!
  }

  const engine = () => {
    frame()
    requestAnimationFrame(engine)
  }

  elSpin?.addEventListener('click', () => {
    if (isSpinning) return
    isSpinning = true
    isAccelerating = true
    angVelMax = rand(0.25, 0.4)
  })

  // INIT!
  sectors.forEach(drawSector)
  rotate() // Initial rotation
  engine() // Start engine!
})
</script>
