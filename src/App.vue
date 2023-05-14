<template>
  <div class="wrapper">
    <Countdown :deadlineISO="now" labelColor="#FFFFFF" />
  </div>
</template>
<script setup lang="ts">
import Countdown from "@/components/Countdown.vue";
import { DateTime } from "luxon";
const now = DateTime.now().plus({ days: 1, hours: 6 }).toISO();
</script>
<style scoped lang="scss">
.wrapper {
  height: 100%;
  width: 100vw;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  padding: 2rem 4rem;
}
</style>
