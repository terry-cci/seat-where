<script setup lang="ts">
import { computed, toRefs } from "vue";
import { StudentCardInfo } from "../App.vue";

const props = defineProps<{ info: StudentCardInfo }>();
const { info } = toRefs(props);

const isPlaced = computed(
  () => info.value.pos.row !== -1 && info.value.pos.col !== -1
);

const gap = 1;
const padding = 2;
</script>

<template>
  <div
    class="student-card border border-gray-400 rounded-sm w-24 h-14 absolute"
    :style="
      isPlaced
        ? {
            left:
              padding +
              (info.pos.col - 1) * 6 +
              Number(info.pos.col >= 3) * gap +
              Number(info.pos.col >= 6) * gap +
              'rem',
            bottom: padding + (info.pos.row - 1) * 3.5 + 'rem',
          }
        : {
            left: '1rem',
            bottom: '1rem',
          }
    "
  >
    <div
      class="
        relative
        front
        flex flex-col
        items-center
        justify-center
        w-full
        h-full
        bg-yellow-50
        text-gray-800
      "
      :style="
        info.flipped
          ? {
              transform: 'rotateX(180deg)',
            }
          : {}
      "
    >
      <span>
        {{ info.name.zh }}
      </span>
      <span class="text-xs text-gray-400">
        {{ info.name.en }} ({{ info.id }})
      </span>
    </div>

    <div
      class="
        back
        absolute
        bg-indigo-500
        text-white
        w-full
        h-full
        top-0
        flex flex-col
        items-center
        justify-center
        text-2xl
        font-bold
      "
      :style="
        info.flipped
          ? {
              transform: 'rotateX(0deg)',
            }
          : {}
      "
    >
      ?
    </div>
  </div>
</template>

<style scoped>
.student-card {
  transition-property: bottom, left;
  transition-duration: 1500ms;
}
.back {
  transform-origin: top;
  transform: rotateX(180deg);
}
.front,
.back {
  backface-visibility: hidden;
  transition-property: transform;
  transition-duration: 150ms;
}
</style>
