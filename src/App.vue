<script setup lang="ts">
import { ref } from "vue";
import StudentCard from "./components/StudentCard.vue";
import s2aStudents from "./data/s2a";
import seedrandom from "seedrandom";

const rng = seedrandom(Date.now().toString() + new Date().toLocaleDateString());

export type Pos = {
  row: number;
  col: number;
};

export interface StudentCardInfo {
  cardId: number;
  id: number;
  name: {
    zh: string;
    en: string;
  };
  pos: Pos;
  flipped: boolean;
}

const studentCardInfo = ref(
  s2aStudents.map((student, i) => ({
    ...student,
    cardId: i,
    pos: { row: -1, col: -1 },
    flipped: true,
  })) as StudentCardInfo[]
);

const interval = 75;
let shuffling = false;

const started = ref(false);

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

  setTimeout(moveCard, 0, 0);
}

function startShuffle() {
  shuffleCards();
}

function moveCard(idx: number) {
  // check if the current card belongs to a fixed student
  const fixedStudent = fixedStudents.find(
    ({ id }) =>
      id === studentCardInfo.value[idx].id && studentCardInfo.value[idx].flipped
  );
  if (fixedStudent) {
    // find the destination student
    const targetStudIdx = studentCardInfo.value.findIndex(
      ({ flipped }, idx) =>
        Math.floor(idx / 7) + 1 === fixedStudent.pos.row &&
        Math.floor(idx % 7) + 1 === fixedStudent.pos.col &&
        flipped
    );
    if (targetStudIdx !== -1) {
      // swap their information excepted card id
      [studentCardInfo.value[idx], studentCardInfo.value[targetStudIdx]] = [
        {
          ...studentCardInfo.value[targetStudIdx],
          pos: studentCardInfo.value[idx].pos,
          cardId: studentCardInfo.value[idx].cardId,
        },
        {
          ...studentCardInfo.value[idx],
          pos: studentCardInfo.value[targetStudIdx].pos,
          cardId: studentCardInfo.value[targetStudIdx].cardId,
        },
      ];
    }
  }

  studentCardInfo.value[idx].pos.row = Math.floor(idx / 7) + 1;
  studentCardInfo.value[idx].pos.col = Math.floor(idx % 7) + 1;

  if (idx + 1 < studentCardInfo.value.length) {
    const moved =
      studentCardInfo.value[idx + 1].pos.row !==
        Math.floor((idx + 1) / 7) + 1 ||
      studentCardInfo.value[idx + 1].pos.col !== Math.floor((idx + 1) % 7) + 1;

    setTimeout(moveCard, moved ? interval : 0, idx + 1);
  } else shuffling = false;
}

function unflip(id: number) {
  studentCardInfo.value[id].flipped = false;
}

// fixing positions
const fixedStudents: { id: number; pos: Pos }[] = [
  {
    id: 16,
    pos: { row: 1, col: 2 },
  },
];

// CYH - Muff fixing
(() => {
  const r = Math.floor(rng() * 4) + 2;
  const c1 = Math.floor(rng() * 7) + 1;
  const c2 = (() => {
    if (c1 <= 2) {
      return c1 === 1 ? 2 : 1;
    }
    if (c1 >= 6) {
      return c1 === 6 ? 7 : 6;
    }
    if (c1 === 4) {
      return [3, 5][Math.floor(rng() * 2)];
    }
    return 4;
  })();

  fixedStudents.push({ id: 18, pos: { row: r, col: c1 } });
  fixedStudents.push({ id: 3, pos: { row: r, col: c2 } });
})();

console.debug(fixedStudents);
</script>

<template>
  <div class="w-full h-full p-2 grid place-items-center">
    <div id="board" class="relative bg-white shadow rounded">
      <student-card
        v-for="(info, idx) in studentCardInfo"
        :key="info.cardId"
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
      @click="startShuffle"
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
