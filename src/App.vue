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
  let c = studentCardInfo.value.findIndex((s) => s.id === 32);
  let m = studentCardInfo.value.findIndex((s) => s.id === 15);

  if (!studentCardInfo.value[c].flipped && !studentCardInfo.value[m].flipped)
    return;

  if (!studentCardInfo.value[m].flipped) {
    const k = c;
    c = m;
    m = k;
  }

  const destOptions = studentCardInfo.value
    .map((card, idx) => ({ idx, card }))
    .filter((v) => v.card.flipped)
    .map((v) => v.idx);

  const dist = (a: number, b: number) => {
    const ax = a % 7;
    const ay = Math.floor(a / 7);
    const bx = b % 7;
    const by = Math.floor(b / 7);
    return Math.max(Math.abs(ax - bx), Math.abs(ay - by));
  };

  let dest = m;
  for (let i = 0; i < destOptions.length; i++) {
    const d = destOptions[i];

    const curDist = dist(d, c);
    const oldDist = dist(dest, c);
    if (oldDist >= 3) break;
    if (curDist > oldDist) dest = d;
  }

  [studentCardInfo.value[m], studentCardInfo.value[dest]] = [
    studentCardInfo.value[dest],
    studentCardInfo.value[m],
  ];
}

function shuffleCards() {
  started.value = true;
  if (shuffling) return;

  const shuffleArrayIdx = studentCardInfo.value
    .map((card, idx) => ({ idx, card }))
    .filter((v) => v.card.flipped)
    .map((v) => v.idx);

  const shuffleArrayHolder = studentCardInfo.value.filter(
    (card) => card.flipped
  );

  let curIdx = shuffleArrayHolder.length;
  let randIdx;

  while (curIdx > 0) {
    curIdx--;
    randIdx = Math.floor(rng() * curIdx);

    [shuffleArrayHolder[curIdx], shuffleArrayHolder[randIdx]] = [
      shuffleArrayHolder[randIdx],
      shuffleArrayHolder[curIdx],
    ];
  }

  shuffleArrayHolder.forEach((card, i) => {
    studentCardInfo.value[shuffleArrayIdx[i]] = card;
  });

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
        @click="unflip(idx)"
        v-show="started"
      />
      <button
        class="
          bg-indigo-400
          absolute
          -bottom-16
          left-1/2
          transform
          -translate-x-1/2
          border
          shadow
          text-white
          rounded
          text-xl
          px-8
          py-2
        "
        @click="shuffleCards"
        v-if="started"
      >
        轉運
      </button>
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
