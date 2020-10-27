<template>
  <!-- <template> -->
  <div id="board">
    <tr v-for="(row, rowIndex) in matrix" :key="`row-${rowIndex}`">
      <td
        v-for="(cell, cellIndex) in row"
        :key="`cell-${rowIndex}-${cellIndex}`"
        :class="{ cell: true, living: cell, dead: !cell }"
      ></td>
    </tr>
  </div>
  <span id="button">
    <br />
    <button @click.prevent="restart">Restart</button>
    <button @click.prevent="stopNow">Stop</button>
  </span>
  <!-- </template> -->
  <!-- <template v-else>
    <h1>Firefox sucks. Open this in Chrome</h1>
  </template> -->
</template>

<script>
import { ref } from 'vue'
const createFilledArray = (size, filler) => Array(size).fill().map(filler)

const initMatrix = ({ height, width, empty = false }) =>
  createFilledArray(height, () =>
    createFilledArray(width, () => (empty ? true : Math.random() > 0.5))
  )

const countLivingNeighbours = (matrix, x, y) =>
  [
    (matrix[x + 1] || [])[y + 1],
    (matrix[x + 1] || [])[y],
    (matrix[x + 1] || [])[y - 1],
    (matrix[x] || [])[y + 1],
    (matrix[x] || [])[y - 1],
    (matrix[x - 1] || [])[y + 1],
    (matrix[x - 1] || [])[y],
    (matrix[x - 1] || [])[y - 1]
  ].reduce((result, cell) => (cell ? result + 1 : result), 0)

const getMatrixWidth = () => Math.floor(window.innerWidth / 10)
const getMatrixHeight = () => Math.floor((window.innerHeight - 50) / 10)

const shouldBecomeAlive = (neighboursCount) => neighboursCount === 3
const shouldStayAlive = (neighboursCount) => [2, 3].includes(neighboursCount)

export default {
  name: 'App',
  setup() {
    const isFirefox = false // navigator.userAgent.toLowerCase().includes('firefox')
    const matrix = ref(
      initMatrix({ width: getMatrixWidth(), height: getMatrixHeight(), empty: true })
    )
    const shouldStop = ref(false)

    return { isFirefox, matrix, shouldStop }
  },
  methods: {
    update() {
      const newMatrix = this.matrix.map((row, x) =>
        row.map((cell, y) => {
          const livingNeighbours = countLivingNeighbours(this.matrix, x, y)
          return cell ? shouldStayAlive(livingNeighbours) : shouldBecomeAlive(livingNeighbours)
        })
      )

      this.matrix = newMatrix

      window.requestAnimationFrame(() => {
        if (!this.shouldStop) this.update()
      })
    },
    start() {
      this.shouldStop = false
      this.update()
    },
    stopNow() {
      this.shouldStop = true
    },
    restart() {
      this.stopNow()

      this.matrix = initMatrix({ width: getMatrixWidth(), height: getMatrixHeight(), empty: true })

      setTimeout(() => {
        this.matrix = initMatrix({ width: getMatrixWidth(), height: getMatrixHeight() })
        setTimeout(() => {
          this.start()
        }, 1000)
      }, 1000)
    }
  },
  mounted() {
    // if (this.isFirefox) return

    this.restart()
  }
}
</script>

<style>
body {
  background-color: black;
  border: 0;
  padding: 0;
  color: white;
}

.cell {
  width: 10px;
  height: 10px;
  border-radius: 10px;
}

.living {
  background-color: lime;
}

.dead {
  background-color: black;
}

td,
tr {
  border: 0;
  margin: 0;
  padding: 0;
}
</style>
