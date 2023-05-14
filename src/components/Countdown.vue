<template>
  <div class="flip-clock" ref="flipclock">
    <template v-for="data in timeData" :key="data.label">
      <span class="flip-clock__piece" :id="data.elementId" v-show="data.show">
        <div v-if="flipAnimation">
          <span
            class="flip-clock__card flip-card"
            :style="countdownSize ? `font-size:${countdownSize}` : ''"
          >
            <b class="flip-card__top"
              ><span>{{ twoDigits(data.current) }}</span></b
            >
            <b
              class="flip-card__bottom"
              :data-value="twoDigits(data.current)"
            ></b>
            <b
              class="flip-card__back"
              :data-value="twoDigits(data.previous)"
            ></b>
            <b
              class="flip-card__back-bottom"
              :data-value="twoDigits(data.previous)"
            ></b>
          </span>
        </div>
        <div v-else>
          <span class="no-animation__card">{{ twoDigits(data.current) }}</span>
        </div>
        <span
          v-if="showLabels"
          class="flip-clock__slot"
          :style="labelSize ? `font-size:${labelSize}` : ''"
          >{{ data.label }}</span
        >
      </span>
    </template>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  computed,
  watch,
  onBeforeMount,
  onMounted,
  onUnmounted,
  type Ref,
} from "vue";
import { v4 as uuidv4 } from "uuid";
import { DateTime } from "luxon";
import { useElementSize } from "@vueuse/core";

interface IProps {
  deadline?: string;
  deadlineISO?: string;
  deadlineDate?: Date;
  countdownSize?: string;
  labelSize?: string;
  stop?: boolean;
  flipAnimation?: boolean;
  showDays?: boolean;
  showHours?: boolean;
  showMinutes?: boolean;
  showSeconds?: boolean;
  showLabels?: boolean;
  labels?: {
    days: string;
    hours: string;
    minutes: string;
    seconds: string;
  };
  mainColor?: string;
  secondFlipColor?: string;
  mainFlipBackgroundColor?: string;
  secondFlipBackgroundColor?: string;
  labelColor?: string;
  timeZone?: string;
}

const props = withDefaults(defineProps<IProps>(), {
  deadline: () => "",
  flipAnimation: () => true,
  showDays: () => true,
  showHours: () => true,
  showMinutes: () => true,
  showSeconds: () => true,
  showLabels: () => true,
  labels: () => {
    return {
      days: "Days",
      hours: "Hours",
      minutes: "Minutes",
      seconds: "Seconds",
    };
  },
  mainColor: () => "#EC685C",
  secondFlipColor: () => "#EC685C",
  mainFlipBackgroundColor: () => "#222222",
  secondFlipBackgroundColor: () => "#393939",
  labelColor: () => "#222222",
  timeZone: () => "America/New_York",
});

// define emits
const emit = defineEmits(["timeElapsed"]);

// Element Size
const flipclock = ref<HTMLElement | null>();
const size = useElementSize(flipclock);

// Const variables
const NOW = DateTime.now().setZone(props.timeZone).toISO();

// Reactive state
const uuid = uuidv4();
const deadline = ref(props.deadline !== "" ? props.deadline : NOW);
const stop = ref(props.stop);
const showDays = ref(props.showDays);
const showHours = ref(props.showHours);
const showMinutes = ref(props.showMinutes);
const showSeconds = ref(props.showSeconds);
const labels = ref(props.labels);
const deadlineDate = ref(props.deadlineDate);
const deadlineISO = ref(props.deadlineISO);

const now = ref(Math.trunc(DateTime.now().toSeconds() / 1000));
const date: Ref<number | null> = ref(null);
const interval: Ref<number | null> = ref(null);
const diff = ref(0);
const show = ref(false);
const timeData = ref([
  {
    current: 0,
    previous: 0,
    label: labels.value.days,
    elementId: "flip-card-days-" + uuid,
    show: showDays.value,
  },
  {
    current: 0,
    previous: 0,
    label: labels.value.hours,
    elementId: "flip-card-hours-" + uuid,
    show: showHours.value,
  },
  {
    current: 0,
    previous: 0,
    label: labels.value.minutes,
    elementId: "flip-card-minutes-" + uuid,
    show: showMinutes.value,
  },
  {
    current: 0,
    previous: 0,
    label: labels.value.seconds,
    elementId: "flip-card-seconds-" + uuid,
    show: showSeconds.value,
  },
]);

// ... (rest of the reactive state declarations)

// Computed properties
const seconds = computed(() => {
  return Math.trunc(diff.value) % 60;
});

const minutes = computed(() => {
  return Math.trunc(diff.value / 60) % 60;
});

const hours = computed(() => {
  return Math.trunc(diff.value / 60 / 60) % 24;
});

const days = computed(() => {
  return Math.trunc(diff.value / 60 / 60 / 24);
});

// Watchers
watch(deadline, (newVal) => {
  if (!newVal) {
    return;
  }
  const endTime = newVal;
  date.value =
    DateTime.fromISO(endTime).setZone(props.timeZone).toSeconds() / 1000;
  if (!date.value) {
    throw new Error("Invalid props value, correct the 'deadline'");
  }
});

watch(now, () => {
  if (date.value === null) {
    return;
  }
  diff.value = date.value - now.value;
  if (diff.value <= 0 || stop.value) {
    diff.value = 0;
    updateTime(3, 0);
  } else {
    updateAllCards();
  }
});

watch(diff, (newVal) => {
  if (newVal == 0) {
    emit("timeElapsed");
    updateAllCards();
  }
});

// watch(size.height, (newHeight) => {
//   const height = newHeight;

//   const flipCards = document.getElementsByClassName("flip-card");
//   const card = flipCards[0] as HTMLElement;
//   if (card) {
//     for (let i = 0; i < card.children.length; i++) {
//       const child = card.children[i] as HTMLElement;
//       if (child.classList[0].includes("top")) {
//         const childSpan = child.children[0] as HTMLElement;
//         childSpan.style.bottom = -1 * (child.clientHeight / 2 - 5) + "px";
//       }
//     }
//   }
// });

// ... (rest of the watchers)

// Functions
const updateAllCards = () => {
  updateTime(0, days.value);
  updateTime(1, hours.value);
  updateTime(2, minutes.value);
  updateTime(3, seconds.value);
};

const twoDigits = (value: number) => {
  if (value != undefined) {
    if (value.toString().length <= 1) {
      return "0" + value.toString();
    }
    return value.toString();
  } else {
    return "00";
  }
};

const updateTime = (currentIndex: number, newTime: number | Ref<number>) => {
  if (currentIndex >= timeData.value.length || newTime === undefined) {
    return;
  }

  const d = timeData.value[currentIndex];
  if (typeof newTime === "number") {
    newTime = ref(newTime);
  }
  const val = newTime.value < 0 ? 0 : newTime.value;
  const element: HTMLElement | null = document.querySelector(`#${d.elementId}`);

  if (val !== d.current) {
    d.previous = d.current;
    d.current = val;

    if (element) {
      element.classList.remove("flip");
      void element.offsetWidth;
      element.classList.add("flip");
    }

    if (currentIndex === 0) {
      const elements = element?.querySelectorAll("span b");
      if (elements) {
        for (let i = 0; i < elements.length; i++) {
          const cls = elements[i].classList[0];
          if (newTime.value / 1000 >= 1) {
            if (!cls.includes("-4digits")) {
              const newClass = cls + "-4digits";
              elements[i].classList.add(newClass);
              elements[i].classList.remove(cls);
            }
          }
        }
      } else {
        // console.log('no elements');
      }
    }
  }
};

const updateNow = () => {
  now.value = Math.trunc(DateTime.now().setZone(props.timeZone).toSeconds());
};

// Lifecycle hooks
onBeforeMount(() => {
  if (!deadline.value) {
    throw new Error("Missing props 'deadline'");
  }
  const endTime = deadline.value;
  let epoch = DateTime.fromISO(endTime).toSeconds();
  if (deadlineDate.value != null) {
    epoch = deadlineDate.value.getSeconds();
  }
  if (deadlineISO.value) {
    epoch = DateTime.fromISO(deadlineISO.value).toSeconds();
  }
  date.value = epoch;
  if (!date.value) {
    throw new Error("Invalid props value, correct the 'deadline'");
  }
  updateNow();
  interval.value = setInterval(() => {
    updateNow();
  }, 1000);
});

onMounted(() => {
  if (diff.value !== 0 && !stop.value) {
    show.value = true;
  }
});

onUnmounted(() => {
  if (interval.value) {
    clearInterval(interval.value);
  }
});
</script>

<style scoped lang="scss">
.no-animation__card {
  font-weight: 500;
  font-size: 1.1cqw;
  line-height: 1.5;
  display: block;
  color: v-bind(mainColor);
}
.flip-clock {
  text-align: center;
  perspective: 600px;
  display: grid;
  grid-template-rows: max-content min-content;
  grid-template-columns: repeat(4, 1fr);
  justify-items: center;
  align-items: center;
  container-size: normal;
  text-align: center;

  *,
  *:before,
  *:after {
    box-sizing: border-box;
  }

  &__piece {
    display: inline-block;
    margin: 0 1cqw;
    width: 100%;
  }

  &__slot {
    font-size: 2cqw;
    line-height: 1.5;
    display: block;
    color: v-bind(labelColor);
    grid-row: 3 / span 1;
  }
}

.flip {
  width: 100%;
}

.flip-card {
  display: grid;
  grid-template-columns: minmax(5vw, 1fr);
  grid-template-rows: repeat(2, minmax(5vw, 1fr));
  justify-content: center;
  position: relative;
  padding-bottom: 2cqh;
  container-type: inline-size;
  font-size: 6cqw;
  line-height: 0.95;
  grid-row: 1 / span 2;

  @media only screen and (min-width: 500px) {
    font-size: 4cqw;
  }

  &__top {
    grid-row: 1 / span 1;
  }

  &__bottom {
    grid-row: 2 / span 1;

    &::before,
    &::after {
      grid-row: 2 / span 1;
    }
  }

  &__back {
    grid-row: 1 / span 1;

    &::before,
    &::after {
      grid-row: 1 / span 1;
    }
  }

  &__back-bottom {
    grid-row: 2 / span 1;

    &::before,
    &::after {
      grid-row: 2 / span 1;
    }
  }

  &__top,
  &__bottom,
  &__back-bottom,
  &__back::before,
  &__back::after {
    display: block;
    color: v-bind(mainColor);
    background: v-bind(mainFlipBackgroundColor);
    padding: 8cqw 2cqw 0;
    border-radius: 0.15em 0.15em 0 0;
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    transform-style: preserve-3d;
    width: 100%;
    height: 100%;
  }

  &__top-4digits,
  &__bottom-4digits,
  &__back-bottom-4digits,
  &__back-4digits::before,
  &__back-4digits::after {
    display: block;
    color: v-bind(mainColor);
    background: v-bind(mainFlipBackgroundColor);
    padding: 8cqw 2cqw 1cqw;
    border-radius: 0.15em 0.15em 0 0;
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    transform-style: preserve-3d;
    width: 100cqw;
    height: 100%;
  }
}

.flip-card__bottom,
.flip-card__back-bottom,
.flip-card__bottom-4digits,
.flip-card__back-bottom-4digits {
  color: v-bind(secondFlipColor);
  height: 100%;
  width: 100%;
  position: absolute;
  border-top: solid 1px #000;
  background: v-bind(secondFlipBackgroundColor);
  border-radius: 0 0 0.15em 0.15em;
  pointer-events: none;
  overflow: hidden;
  z-index: 2;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
  grid-row: 2 / span 1;
  justify-content: center;
}

.flip-card__back-bottom,
.flip-card__back-bottom-4digits {
  z-index: 1;
}

.flip-card__bottom::after,
.flip-card__back-bottom::after,
.flip-card__bottom-4digits::after,
.flip-card__back-bottom-4digits::after {
  display: block;
  margin-top: -4.5cqh;
}

.flip-card__back::before,
.flip-card__bottom::after,
.flip-card__back-bottom::after,
.flip-card__back-4digits::before,
.flip-card__bottom-4digits::after,
.flip-card__back-bottom-4digits::after {
  content: attr(data-value);
}

.flip-card__back,
.flip-card__back-4digits {
  height: 100%;
  width: 100%;
  position: absolute;
  pointer-events: none;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
  justify-content: center;
}

.flip-card__back::before,
.flip-card__back-4digits::before {
  position: relative;
  overflow: hidden;
  z-index: -1;
}

.flip .flip-card__back::before,
.flip .flip-card__back-4digits::before {
  z-index: 1;
  animation: flipTop 0.3s cubic-bezier(0.37, 0.01, 0.94, 0.35);
  animation-fill-mode: both;
  transform-origin: center bottom;
}

.flip .flip-card__bottom,
.flip .flip-card__bottom-4digits {
  transform-origin: center top;
  animation-fill-mode: both;
  animation: flipBottom 0.6s cubic-bezier(0.15, 0.45, 0.28, 1);
}

@keyframes flipTop {
  0% {
    transform: rotateX(0deg);
    z-index: 2;
  }

  0%,
  99% {
    opacity: 1;
  }

  100% {
    transform: rotateX(-90deg);
    opacity: 0;
  }
}

@keyframes flipBottom {
  0%,
  50% {
    z-index: -1;
    transform: rotateX(90deg);
    opacity: 0;
  }

  51% {
    opacity: 1;
  }

  100% {
    opacity: 1;
    transform: rotateX(0deg);
    z-index: 5;
  }
}
</style>
