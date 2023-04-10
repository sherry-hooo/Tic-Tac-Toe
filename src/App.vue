<script setup>
import { ref, computed } from "vue";
const XAxis = ["a", "b", "c"];
const YAxis = ["1", "2", "3"];
const rowWidth = computed(() => `${XAxis.length * 100 + XAxis.length}px`);
const players = ["o", "x"];
const currentPlayerNo = ref(0);
const msg = ref("Start");
const ifAnyWin = computed(() => hasWinner(allCols.value));

const allCols = ref(
  Array(YAxis.length * XAxis.length)
    .fill(1)
    .map((col, colIndex) => {
      const colSeq = colIndex + col;
      const y = Math.floor(colSeq / XAxis.length);
      const remaining = colSeq % XAxis.length;
      return {
        id: colSeq,
        x: remaining === 0 ? XAxis[y - 1] : XAxis[y],
        y: remaining === 0 ? YAxis[XAxis.length - 1] : YAxis[remaining - 1],
      };
    })
);

const isMinSelected = (filledCols) => filledCols.length >= XAxis.length;
const hasWinner = (allFilledCols) => {
  const filledCols = allFilledCols.filter(
    (col) => col.selectedBy === players[currentPlayerNo.value]
  );
  if (!isMinSelected(filledCols)) return;
  return checkXYAxis(filledCols) || checkDiagonal(filledCols);
};

function handlePlay(targetCol) {
  // return 已經選過的格子
  if (targetCol.selectedBy || ifAnyWin.value) return;

  const markedCol = {
    ...targetCol,
    selectedBy: players[currentPlayerNo.value],
  };
  allCols.value = allCols.value.map((oriCol) => {
    return oriCol.x === targetCol.x && oriCol.y === targetCol.y
      ? markedCol
      : oriCol;
  });

  if (ifAnyWin.value) {
    msg.value = `${markedCol.selectedBy} win!!!`;
  } else {
    // 沒有輸贏繼續下一局
    currentPlayerNo.value = goNextPlayer(markedCol.selectedBy);
    msg.value = `${players[currentPlayerNo.value]} turn`;
  }
}

function restart() {
  currentPlayerNo.value = 0;
  msg.value = "Start";
  allCols.value = allCols.value.map((col) => ({ ...col, selectedBy: "" }));
}

function getDiagonalCols(idCols) {
  const leftDiagonal = [];
  const rightDiagonal = [];
  for (
    let leftIdx = 0, rightIdx = XAxis.length - 1;
    leftIdx <= 9;
    leftIdx += XAxis.length + 1, rightIdx += XAxis.length - 1
  ) {
    leftDiagonal.push(idCols[leftIdx]);
    rightDiagonal.push(idCols[rightIdx]);
  }

  return [leftDiagonal, rightDiagonal];
}
const diagonalWinningCols = getDiagonalCols(allCols.value.map((col) => col.id));

function checkDiagonal(cols) {
  return diagonalWinningCols.some((winningCols) =>
    winningCols.every((number) => cols.map((col) => col.id).includes(number))
  );
}
function checkXYAxis(cols) {
  const initial = [...XAxis, ...YAxis].reduce((acc, lineSymbol) => {
    if (lineSymbol in acc) {
      return {
        ...acc,
        [lineSymbol]: acc[lineSymbol] + 1,
      };
    } else {
      return { ...acc, [lineSymbol]: 0 };
    }
  }, {});

  const calResult = cols.reduce((result, selectedCol) => {
    result = {
      ...result,
      [selectedCol.x]: result[selectedCol.x] + 1,
      [selectedCol.y]: result[selectedCol.y] + 1,
    };
    return result;
  }, initial);
  return Object.values(calResult).includes(XAxis.length);
}

function goNextPlayer(currentPlayer) {
  const maxIndex = players.length - 1;
  4;
  const currentIndex = players.findIndex((player) => player === currentPlayer);
  return currentIndex === maxIndex ? 0 : currentIndex + 1;
}
</script>

<template>
  <div class="tic-tac-toe">
    <h1>Tic-Tac-Toe</h1>
    <p>{{ msg }}</p>

    <section class="container">
      <div
        class="col"
        v-for="col in allCols"
        :key="col.x + col.y"
        @click="(e) => handlePlay(col)"
      >
        <span class="symbol">
          {{ col.selectedBy }}
        </span>
      </div>
    </section>

    <button @click="restart">Restart</button>
  </div>
</template>

<style scoped>
.tic-tac-toe {
  margin: auto auto;
}

.container {
  display: flex;
  flex-wrap: wrap;
  width: v-bind(rowWidth);
  gap: 1px;
  margin: 20px 0;
}
.col {
  flex: 1 1 auto;
  width: 100px;
  height: 100px;
  background: white;
  color: black;
  text-align: center;
  position: relative;
  cursor: pointer;
}
.col:hover {
  background: rgb(244, 205, 78);
}
.hint {
  position: absolute;
  top: 1px;
  left: 5px;
}

.symbol {
  line-height: 100px;
  vertical-align: middle;
  font-size: 20px;
  font-weight: 700;
}
</style>
