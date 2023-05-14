<template>
  <svg
    xmlns="http://www.w3.org/2000/svg"
    class="responsive-svg"
    viewBox="0 0 25 25"
    preserveAspectRatio="xMinYMin slice"
    ref="svg"
  >
    <text
      x="50%"
      y="50%"
      font-family="sans-serif"
      fill="currentColor"
      dominant-baseline="middle"
      text-anchor="middle"
      ref="text"
    >
      {{ number }}
    </text>
  </svg>
</template>
<script setup lang="ts">
import { v4 as uuidv4 } from "uuid";
import { onMounted, ref, watch } from "vue";

const props = defineProps({
  number: {
    type: Number,
    required: true,
  },
});

const uuid = uuidv4();
const svg = ref<SVGElement | undefined>();
const text = ref<SVGTextElement | undefined>();

const viewBox = ref("0 0 20 20");
const textX = ref(50);
const textY = ref(50);

// watch(window, (windowNew) => {
//   if (svg.value && windowNew) {
//     if (window.innerHeight !== windowNew.innerHeight) {
//       console.log("Height changed");
//     } else if (window.innerWidth !== windowNew.innerWidth) {
//       console.log("Width changed");
//     }
//   }
// });

// onMounted(() => {
//   if (svg.value && text.value) {
//     const svgWidth = svg.value.clientWidth;
//     const svgHeight = svg.value.clientHeight;
//     const bbox = text.value.getBBox();
//     viewBox.value = `${bbox.x} ${bbox.y} ${svgWidth} ${svgHeight}`;
//     console.log(
//       `Viewbox: ${viewBox.value} -- svgWidth/Height: ${svgWidth}/${svgHeight}`
//     );
//     textX.value = bbox.width / 2;
//     textY.value = bbox.height / 2;
//   }
// });
</script>
<style scoped>
.responsive-svg {
  width: 100%;
  height: 100%;
}
</style>
