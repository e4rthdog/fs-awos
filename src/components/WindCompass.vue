<template>
  <!-- SVG compass for wind/runway visualization -->
  <svg
    class="w-full h-full"
    viewBox="-40 -40 480 480"
    xmlns="http://www.w3.org/2000/svg"
    preserveAspectRatio="xMidYMid meet"
  >
    <!-- Compass background -->
    <circle cx="200" cy="200" r="150" fill="#1e1e1e" stroke="#444" stroke-width="4" />
    <!-- Major tick marks -->
    <g stroke="#888" stroke-width="2">
      <g v-for="angle in majorAngles" :key="'tick-' + angle">
        <line x1="200" y1="50" x2="200" y2="60" :transform="'rotate(' + angle + ' 200 200)'" />
      </g>
    </g>
    <!-- Degree labels -->
    <g
      font-size="12"
      fill="#d1d5db"
      font-family="'Share Tech Mono', monospace"
      style="letter-spacing: 3px"
    >
      <text
        v-for="angle in majorAngles"
        :key="'label-' + angle"
        :x="labelX(angle)"
        :y="labelY(angle)"
        text-anchor="middle"
        alignment-baseline="middle"
      >
        {{ angle.toString().padStart(3, '0') }}
      </text>
    </g>
    <!-- Runway centerline and box -->
    <g :transform="'rotate(' + runwayHeading + ' 200 200)'">
      <rect x="185" y="100" width="30" height="200" fill="black" />
      <line
        x1="200"
        y1="105"
        x2="200"
        y2="295"
        stroke="white"
        stroke-width="1.5"
        stroke-dasharray="15, 10"
      />
      <line x1="186" y1="100" x2="186" y2="300" stroke="white" stroke-width="1" />
      <line x1="214" y1="100" x2="214" y2="300" stroke="white" stroke-width="1" />
    </g>
    <!-- Wind direction arrow -->
    <g :transform="'rotate(' + windDirection + ' 200 200)'">
      <polygon points="200,40 190,20 210,20" fill="red" stroke="black" stroke-width="1" />
    </g>
    <!-- Opposite runway marker -->
    <g :transform="'rotate(' + (runwayHeading + 180) + ' 200 200)'">
      <g transform="translate(0, 180)">
        <!-- Use external plane.svg with yellow border, rotated 90deg -->
        <g transform="rotate(90 200 15)">
          <image x="185" y="0" width="30" height="30" xlink:href="/src/assets/plane.svg" />
          <rect x="185" y="0" width="30" height="30" fill="none" />
        </g>
      </g>
    </g>
    <!-- Digital readouts -->
    <g font-size="14" font-family="'Share Tech Mono', monospace" fill="#d1d5db">
      <text
        x="200"
        y="400"
        font-size="18"
        text-anchor="middle"
        fill="#d1d5db"
        font-family="'Share Tech Mono', monospace"
      >
        RWY: {{ runwayName }} - {{ numericWindDirection.toString().padStart(3, '0') }}° /
        {{ windSpeed }}{{ gustDisplay }} kt
      </text>
      <text x="200" y="445" text-anchor="middle">Headwind: {{ headwind.toFixed(1) }} kt</text>
      <text x="200" y="465" text-anchor="middle">
        Crosswind: {{ crosswind.toFixed(1) }} kt {{ crosswindDir }}
      </text>
      <text x="200" y="420" text-anchor="middle" font-size="11" fill="orange" v-if="crosswind > 0">
        {{ crosswindWarning }}
      </text>
    </g>
  </svg>
</template>

<script setup>
import { computed } from 'vue'

// Props for runway and wind data
const props = defineProps({
  runwayHeading: { type: Number, required: true },
  runwayName: { type: String, required: true },
  windDirection: { type: Number, required: true },
  windSpeed: { type: Number, required: true },
  windGust: { type: Number, default: null },
})

// Major compass angles (every 30°)
const majorAngles = computed(() => Array.from({ length: 12 }, (_, i) => i * 30))
const toRadians = (deg) => (deg * Math.PI) / 180
// Relative wind/runway angle for calculations
const relativeAngle = computed(() => {
  let delta = numericWindDirection.value - props.runwayHeading
  delta = ((delta + 540) % 360) - 180
  return delta
})
// Handle VRB wind as 0°
const numericWindDirection = computed(() => {
  if (props.windDirection === 'VRB') return 0
  return Number(props.windDirection) || 0
})
// Headwind/crosswind components
const headwind = computed(() => props.windSpeed * Math.cos(toRadians(relativeAngle.value)))
const crosswind = computed(() =>
  Math.abs(props.windSpeed * Math.sin(toRadians(relativeAngle.value))),
)
// Crosswind direction (left/right)
const crosswindDir = computed(() => {
  const dir = relativeAngle.value
  if (dir === 0 || dir === 180) return ''
  return dir > 0 ? '(from right)' : '(from left)'
})
// Gust display logic
const hasGust = computed(() => props.windGust && props.windGust > props.windSpeed)
const gustDisplay = computed(() => (hasGust.value ? `G${props.windGust}` : ''))
// Crosswind warning text
const crosswindWarning = computed(() => {
  const xwind = crosswind.value
  if (xwind > 25) return 'Severe crosswind conditions'
  if (xwind > 15) return 'Strong crosswind conditions'
  if (xwind > 10) return 'Moderate crosswind conditions'
  if (xwind > 5) return 'Light crosswind conditions'
  return ''
})
// Label positions for compass
const labelRadius = 123
const labelX = (angle) => {
  const rad = toRadians(angle)
  return 200 + labelRadius * Math.sin(rad)
}
const labelY = (angle) => {
  const rad = toRadians(angle)
  return 200 - labelRadius * Math.cos(rad)
}
</script>

<style scoped>
svg {
  display: block;
  width: 100%;
  height: 100%;
  overflow: visible;
}
</style>
