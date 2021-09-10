<script setup lang="ts">
import { ref } from "vue";
import StudentCard from "./components/StudentCard.vue";
import s2aStudents from "./data/s2a";
import seedrandom from "seedrandom";

const rng = seedrandom(Date.now().toString());

export interface StudentCardInfo {
  id: number;
  name: {
    zh: string;
    en: string;
  };
  pos: {
    row: number;
    col: number;
  };
  flipped: boolean;
}

const studentCardInfo = ref(
  s2aStudents.map((student) => ({
    ...student,
    pos: { row: -1, col: -1 },
    flipped: true,
  })) as StudentCardInfo[]
);

const interval = 75;
let shuffling = false;

const started = ref(false);

function escape() {
  const c = studentCardInfo.value.findIndex((s) => s.id === 32);
  let m = studentCardInfo.value.findIndex((s) => s.id === 15);
  const v = m > c ? 1 : -1;

  const dist = () => {
    const mx = m % 7;
    const my = Math.floor(m / 7);
    const cx = c % 7;
    const cy = Math.floor(c / 7);
    return Math.max(Math.abs(mx - cx), Math.abs(my - cy));
  };

  while (dist() < 3) {
    console.debug(m);
    const d = (m + v + 35) % 35;
    [studentCardInfo.value[m], studentCardInfo.value[d]] = [
      studentCardInfo.value[d],
      studentCardInfo.value[m],
    ];
    m = d;
  }
}

function shuffleCards() {
  started.value = true;
  if (shuffling) return;

  let curIdx = studentCardInfo.value.length;
  let randIdx;

  while (curIdx > 0) {
    randIdx = Math.floor(rng() * curIdx);
    curIdx--;

    [studentCardInfo.value[curIdx], studentCardInfo.value[randIdx]] = [
      studentCardInfo.value[randIdx],
      studentCardInfo.value[curIdx],
    ];
  }

  shuffling = true;

  escape();

  setTimeout(moveCard, 0, 0);
}

function moveCard(id: number) {
  studentCardInfo.value[id].pos.row = Math.floor(id / 7) + 1;
  studentCardInfo.value[id].pos.col = Math.floor(id % 7) + 1;

  if (id + 1 < studentCardInfo.value.length)
    setTimeout(moveCard, interval, id + 1);
  else shuffling = false;
}

function unflip(id: number) {
  studentCardInfo.value[id].flipped = false;
}
const debug = console.debug;
</script>

<template>
  <div class="w-full h-full p-2 grid place-items-center">
    <div id="board" class="relative bg-white shadow rounded">
      <student-card
        v-for="(info, idx) in studentCardInfo"
        :key="info.id"
        :info="info"
        @mouseover="unflip(idx)"
        @click="unflip(idx)"
        v-show="started"
      />
    </div>
    <button
      class="
        fixed
        transform
        -translate-x-1/2 -translate-y-1/2
        top-1/2
        left-1/2
        bg-indigo-400
        border
        shadow
        text-white
        rounded
        text-2xl
        px-16
        py-4
      "
      @click="shuffleCards"
      v-if="!started"
    >
      點擊開始抽取座位
    </button>
  </div>
</template>

<style scoped>
#board {
  height: 21.5rem;
  width: 48rem;
}
</style>
