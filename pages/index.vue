<script setup lang="ts">
import IND from '@/public/ind.json'
const RUN_BOARDS = [
  [null, null, null, null, null],
  [null, null, null, null, null],
  [null, null, null, null, null],
  [null, null, null, null, null],
  [null, null, null, null, null],
]
const VOCALS = ['A', 'I', 'U', 'E', 'O']

const disableBoard = ref(false)
const disableAll = ref(false)
const reRender = ref(true)
const isValidBoard = ref(true)
const isAlreadySolved = ref(false)

const keyDic = ref('IND')
const DIC = ref({
  IND,
})

const row = ref(5)
const col = ref(5)

const boards = ref([
  ['Z', 'K', 'Z', 'Z', 'Z'],
  ['U', 'B', 'U', 'Z', 'Z'],
  ['Z', 'Z', 'Z', 'B', 'Z'],
  ['Z', 'Z', 'A', 'Z', 'A'],
  ['Z', 'Z', 'Z', 'C', 'Z'],
])

const runBoards = ref(JSON.parse(JSON.stringify(RUN_BOARDS)))
const results = ref<any[]>([])
const numbering = ref(1)
const activeResult = ref(0)

function filter({ i, j, e }) {
  if (e.data) {
    // e.data         => a  a    |   1   6
    // e.target.value => a  aa   |   1   6
    // *number selalu 1
    const code = e.data.charCodeAt()
    if ((code >= 65 && code <= 90) || (code >= 97 && code <= 122)) {
      let data = e.target.value
      data = data[data.length - 1]
      data = `${data}`.toUpperCase()
      boards.value[i][j] = data
      e.target.value = data
    } else {
      boards.value[i][j] = ''
      e.target.value = ''
    }
  }
}
function createBoard() {
  disableBoard.value = false
  isAlreadySolved.value = false
  results.value = []
  const _boards: any = []
  const rows: (null | string)[] = []
  for (let i = 1; i <= +col.value; i++) {
    rows.push(null)
  }

  for (let i = 1; i <= +row.value; i++) {
    _boards.push(rows)
  }
  boards.value = JSON.parse(JSON.stringify(_boards))
  runBoards.value = JSON.parse(JSON.stringify(_boards))
}
function checkBoard() {
  isValidBoard.value = true
  for (let i = 0; i < row.value - 1; i++) {
    let isBreakI = false
    for (let j = 0; j < col.value - 1; j++) {
      if (!boards.value[i][j]) {
        isBreakI = true
        isValidBoard.value = false
        break
      }
    }
    if (isBreakI) break
  }
}
function shake() {
  isAlreadySolved.value = false
  results.value = []
  isValidBoard.value = true
  for (let i = 0; i < row.value; i++) {
    for (let j = 0; j < col.value; j++) {
      if (Math.random() * 5 > 1) {
        boards.value[i][j] = String.fromCharCode(
          Math.floor(Math.random() * 26 + 65),
        )
      } else boards.value[i][j] = VOCALS[Math.floor(Math.random() * 5)]
    }
  }
  runBoards.value = JSON.parse(JSON.stringify(boards.value))
  reRender.value = false
  nextTick(() => {
    reRender.value = true
  })
}
function getNextLetter(w, word) {
  if (word[w + 1]) return word[w + 1]
  return null
}
function getMove({ i, j, nextLetter, boards }) {
  //  1 | 2 | 3
  //  8 | X | 4
  //  7 | 6 | 5
  const arrowLabel = {
    1: 'top-left',
    2: 'top',
    3: 'top-right',
    4: 'right',
    5: 'bottom-right',
    6: 'bottom',
    7: 'bottom-left',
    8: 'left',
  }
  const arrowSymbol = {
    1: '↖',
    2: '↑',
    3: '↗',
    4: '→',
    5: '↘',
    6: '↓',
    7: '↙',
    8: '←',
  }
  const arrow = {
    1: { i: -1, j: -1 },
    2: { i: -1, j: 0 },
    3: { i: -1, j: 1 },
    4: { i: 0, j: 1 },
    5: { i: 1, j: 1 },
    6: { i: 1, j: 0 },
    7: { i: 1, j: -1 },
    8: { i: 0, j: -1 },
  }

  const _result: any = { x: i, y: j, nextLetter, flag: false }
  for (const [key, value] of Object.entries(arrow)) {
    const x = i + value.i
    const y = j + value.j
    if (x >= 0 && x < row.value && y >= 0 && y < col.value) {
      if (
        boards[x][y] !== null &&
        !+boards[x][y] &&
        typeof boards[x][y] !== 'object'
      ) {
        if (boards[x][y] === nextLetter) {
          // boards[x][y] = true;
          boards[x][y] = {
            label: boards[x][y],
            step: numbering.value,
          }
          numbering.value++
          boards[i][j].arrow = arrow[key]
          boards[i][j].arrowLabel = arrowLabel[key]
          boards[i][j].arrowSymbol = arrowSymbol[key]

          _result.x = x
          _result.y = y
          _result.flag = true
          break
        }
        // else boards[x][y] = null;
      }
    }
  }
  _result.boards = boards
  return _result
}
function checkWord(word) {
  for (let i = 0; i < row.value; i++) {
    let isAlreadyGetResult = false

    for (let j = 0; j < col.value; j++) {
      const _histories: any[] = []
      let w = 0
      let _boards = JSON.parse(JSON.stringify(boards.value))
      numbering.value = 1

      if (
        _boards[i][j] !== null &&
        !+_boards[i][j] &&
        typeof _boards[i][j] !== 'object'
      ) {
        if (_boards[i][j] === word[w]) {
          _histories.push({ iH: i, jH: j, vH: _boards[i][j] })
          _boards[i][j] = {
            label: _boards[i][j],
            step: numbering.value,
          }
          numbering.value++
          let nextLetter = getNextLetter(w, word)

          if (nextLetter) {
            w++
            let x = i
            let y = j
            let flag = true
            while (flag) {
              const move = getMove({ i: x, j: y, nextLetter, boards: _boards })
              flag = move.flag
              _boards = move.boards

              if (flag) {
                x = move.x
                y = move.y
                _histories.push({ iH: x, jH: y, vH: nextLetter })
                nextLetter = getNextLetter(w, word)
                if (nextLetter) {
                  w++
                } else {
                  results.value.push({
                    word,
                    boards: JSON.parse(JSON.stringify(_boards)),
                  })
                  isAlreadyGetResult = true
                  flag = false
                }
              } else if (_histories.length > 1) {
                const back = _histories.pop()
                _boards[back.iH][back.jH] = null
                const { iH, jH } = _histories.pop()
                x = iH
                y = jH
                w--
                nextLetter = word[w]
                flag = true
              }
            }
          }
        }
      }
      if (isAlreadyGetResult) break
    }
    if (isAlreadyGetResult) break
  }
}
async function solve() {
  disableBoard.value = true
  disableAll.value = true
  await $delay(200)
  await checkBoard()
  if (!isValidBoard.value) {
    disableBoard.value = false
    disableAll.value = false
    return
  }

  isAlreadySolved.value = false
  results.value = []

  const keyOfBoards = {}
  boards.value.forEach((e) => e.forEach((f) => (keyOfBoards[f] = null)))

  for (const key in keyOfBoards) {
    const words = DIC.value[keyDic.value][key]
    for (let w = 0; w < words.length; w++) {
      await checkWord(words[w])
    }
  }
  await $delay()
  disableAll.value = false
  isAlreadySolved.value = true

  if (results.value.length) {
    runBoards.value = results.value[activeResult.value].boards
  }
}
</script>

<template>
  <div class="game">
    <div class="game__header">
      <div class="game__header__title">Boggle Solver</div>
      <div class="game__header__sub-title">(Indonesian word by KBBI)</div>
    </div>

    <div class="game__action">
      <button
        class="btn btn-primary !w-[140px]"
        :disabled="disableAll"
        @click="shake()"
      >
        Auto puzzle
      </button>
      <button
        class="btn btn-primary !w-[140px]"
        :disabled="disableAll"
        @click="createBoard()"
      >
        Manual puzzle
      </button>
    </div>

    <!-- BOARDS -->
    <table v-if="reRender">
      <tr v-for="(rows, i) in boards" :key="i">
        <td v-for="(val, j) in rows" :key="j">
          <!-- S T E P -->
          <span
            v-if="
              runBoards[i][j] !== null && typeof runBoards[i][j] === 'object'
            "
            class="step-info"
          >
            {{ runBoards[i][j].step }}
          </span>

          <!-- A R R O W -->
          <span
            v-if="
              runBoards[i][j] !== null && typeof runBoards[i][j] === 'object'
            "
            :class="`step-arrow step-${runBoards[i][j].arrowLabel}`"
          >
            {{ runBoards[i][j].arrowSymbol }}
          </span>

          <!-- L A B E L -->
          <input
            v-model="boards[i][j]"
            :disabled="disableBoard || disableAll"
            :style="{
              color:
                runBoards[i][j] !== null && typeof runBoards[i][j] === 'object'
                  ? 'var(--c-bootstrap-white)'
                  : '',
            }"
            @input="filter({ i, j, e: $event })"
          />
        </td>
      </tr>
    </table>
    <!-- END  BOARDS -->

    <!-- ACTION -->
    <div>
      <div v-if="!isValidBoard" class="text-center text-sm italic text-red-600">
        * Board tidak valid, silahkan dilengkapi
      </div>
      <button
        class="btn btn-primary btn-sm mt-5"
        :disabled="disableAll"
        @click="solve()"
      >
        Solve
      </button>
      <div v-if="disableAll" class="spinner">
        <div class="rect1"></div>
        <div class="rect2"></div>
        <div class="rect3"></div>
        <div class="rect4"></div>
        <div class="rect5"></div>
      </div>
    </div>
    <!-- END  ACTION -->

    <!-- RESULT -->
    <div v-if="isAlreadySolved" style="margin-top: 1em">
      <div
        v-if="results.length === 0"
        style="font-weight: bold; color: var(--c-bootstrap-warning)"
      >
        there are no words to match
      </div>
      <div v-else>
        <div style="font-weight: bold">
          Found {{ results.length }} {{ results.length > 1 ? 'words' : 'word' }}
        </div>
        <div v-for="(word, i) in results" :key="i">
          <span
            :style="{
              margin: '0.5em 0em',
              cursor: 'pointer',
              'font-weight': activeResult === i ? 'bold' : '',
              color: activeResult === i ? 'var(--c-bootstrap-info)' : '',
            }"
            @click.stop="
              () => {
                runBoards = word.boards
                activeResult = i
              }
            "
          >
            {{ word.word }}
          </span>
        </div>
      </div>
    </div>
    <!-- END  RESULT -->
  </div>
</template>

<style lang="postcss" scoped>
.game {
  @apply mt-[40px];

  &__header {
    @apply text-center;

    &__title {
      @apply font-bold text-2xl;
    }

    &__sub-title {
      @apply mt-2 mb-4;
      @apply font-bold;
    }
  }
  &__action {
    @apply flex justify-between gap-4;
    @apply mt-12 mb-5;
  }
}

table {
  display: inline-block;
  border: solid 0.3em #baada0;
}

table tr td {
  @apply size-[55px];
  @apply bg-[#cdc0b4];
  @apply text-lg text-[#776e65] font-bold text-center;
  @apply border-[2px] border-[#baada0];
}

table tr td input {
  @apply size-[55px];
  @apply bg-[#cdc0b4];
  @apply text-center font-bold text-lg text-[#776e65];
  border: unset !important;
}

table tr td input:focus {
  outline: none;
}

.step {
  &-arrow {
    position: absolute;
    color: var(--c-bootstrap-cyan);
  }

  &-info {
    position: absolute;
    color: var(--c-bootstrap-gray-dark);
    font-size: x-small;
    margin-left: 2px;
    font-weight: bold;
  }

  /*    1 | 2 | 3    */
  /*    8 | X | 4    */
  /*    7 | 6 | 5    */
  &-top-left {
    margin-top: -16px;
    margin-left: -12px;
  }
  &-top {
    margin-top: -16px;
    margin-left: 11px;
  }
  &-top-right {
    margin-top: -14px;
    margin-left: 32px;
  }
  &-right {
    margin-left: 32px;
    margin-top: 6px;
  }
  &-bottom-right {
    margin-top: 26px;
    margin-left: 30px;
  }
  &-bottom {
    margin-top: 30px;
    margin-left: 11px;
  }
  &-bottom-left {
    margin-top: 28px;
    margin-left: -8px;
  }
  &-left {
    margin-left: -12px;
    margin-top: 6px;
  }
}

.btn {
  @apply w-[100%] inline-block;
  @apply text-lg text-center text-nowrap;
  @apply py-[6px] px-[10px] rounded;
  @apply cursor-pointer;
  vertical-align: middle;
  -ms-touch-action: manipulation;
  touch-action: manipulation;
  border: 1px solid transparent;
  line-height: 1.42857143;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.btn.focus,
.btn:focus,
.btn:hover {
  @apply text-[#333];
  text-decoration: none;
}

.btn.active,
.btn:active {
  background-image: none;
  outline: 0;
  -webkit-box-shadow: inset 0 3px 5px rgba(0, 0, 0, 0.125);
  box-shadow: inset 0 3px 5px rgba(0, 0, 0, 0.125);
}

.btn.disabled,
.btn[disabled],
fieldset[disabled] .btn {
  @apply cursor-not-allowed;
  filter: alpha(opacity=65);
  opacity: 0.65;
  -webkit-box-shadow: none;
  box-shadow: none;
}

a.btn.disabled,
fieldset[disabled] a.btn {
  pointer-events: none;
}

.btn-primary {
  @apply bg-slate-100 text-gray-800 border-blue-100;
}

.btn-primary:hover {
  @apply bg-blue-100 text-gray-800 border-blue-400;
}

.spinner {
  margin: 100px auto;
  width: 50px;
  height: 40px;
  text-align: center;
  font-size: 10px;
}

.spinner > div {
  background-color: #333;
  height: 100%;
  width: 6px;
  display: inline-block;

  -webkit-animation: sk-stretchdelay 1.2s infinite ease-in-out;
  animation: sk-stretchdelay 1.2s infinite ease-in-out;
}

.spinner .rect2 {
  -webkit-animation-delay: -1.1s;
  animation-delay: -1.1s;
}

.spinner .rect3 {
  -webkit-animation-delay: -1s;
  animation-delay: -1s;
}

.spinner .rect4 {
  -webkit-animation-delay: -0.9s;
  animation-delay: -0.9s;
}

.spinner .rect5 {
  -webkit-animation-delay: -0.8s;
  animation-delay: -0.8s;
}

@-webkit-keyframes sk-stretchdelay {
  0%,
  40%,
  100% {
    -webkit-transform: scaleY(0.4);
  }
  20% {
    -webkit-transform: scaleY(1);
  }
}

@keyframes sk-stretchdelay {
  0%,
  40%,
  100% {
    transform: scaleY(0.4);
    -webkit-transform: scaleY(0.4);
  }
  20% {
    transform: scaleY(1);
    -webkit-transform: scaleY(1);
  }
}
</style>
