<template>
  <section>
    <h1 class="game--title">
      Tic Tac Toe
    </h1>
    <div v-show="startGame">
      <div id="content" class="game--content" />
      <h2 class="game--status" />
      <button class="game--restart">
        Restart Game
      </button>
      <button class="game--replay">
        Replay Game
      </button>
    </div>
    <div v-show="!startGame">
      <input v-model="ROW_SIZE" type="number">
      <button @click="submit">
        Submit
      </button>
    </div>
    <div v-for="replay in replays" :key="replay.id">
      <!-- <a href="http://localhost:8000/api/replay">Replay: {{ replay.id }}</a> -->
      <NuxtLink :to="`replay/${replay.id}`">
        Replay: {{ replay.id }}
      </NuxtLink>
    </div>
  </section>
</template>

<script>
export default {
  data () {
    return {
      winner: '',
      logs: {
        winner: '',
        log: [],
        row: ''
      },
      gameActive: true,
      ROW_SIZE: 3,
      gameState: [],
      currentPlayer: 'X',
      statusDisplay: '',
      winningConditions: [],
      startGame: false,
      replays: []
    }
  },
  mounted () {
    this.start()
    this.fetch()
  },

  methods: {
    getData () {
      return {
        data: {}
      }
    },
    async fetch () {
      const replays = await this.$axios.$get('http://localhost:8000/api/replay')
      this.replays = replays.data
      console.log(this.replays)
    },

    submit () {
      this.ROW_SIZE = parseInt(this.ROW_SIZE)
      if (this.ROW_SIZE === 0 || this.ROW_SIZE < 3 || this.ROW_SIZE > 8) {
        this.gamestart = false
        alert('In put number between 3 to 8')
        this.ROW_SIZE = 0
      } else {
        this.startGame = true
        this.start()
      }
    },
    getPlayer () {
      if (this.currentPlayer === 'X') {
        this.currentPlayer = 'O'
        return this.currentPlayer
      } else {
        this.currentPlayer = 'X'
        return this.currentPlayer
      }
    },
    getGameState () {
      const state = []
      for (let i = 0; i < this.ROW_SIZE ** 2; i++) {
        state.push('')
      }
      this.gameState = state
      return this.gameState
    },
    getAcross (size) {
      const across = []
      for (let i = size - size; i < size; i++) {
        const dataAcross = []
        for (let j = size - size; j < (size * i) + size; j++) {
          dataAcross.push(j)
        }
        dataAcross.reverse()
        dataAcross.length = size
        dataAcross.sort()
        across.push(dataAcross)
      }

      return across
    },
    getDown (size) {
      const dataDown = []
      for (let i = (size - size); i < size; i++) {
        dataDown.push(i * size)
      }
      const down = []

      for (let i = size - size; i < size; i++) {
        down.push(dataDown.map(value => value + (1 * i)))
      }

      return down
    },
    getcCossLeft (size) {
      const crossLeft = []
      for (let j = size - 1; j < size ** 2 - 1; j += size - 1) {
        crossLeft.push(j)
      }
      return crossLeft
    },
    getcCossRight (size) {
      const crossRight = []
      for (let i = size - size; i < size; i++) {
        crossRight.push(i * (size + 1))
      }
      return crossRight
    },
    getWinningConditions (size) {
      const across = this.getAcross(size)
      const down = this.getDown(size)
      const crossLeft = this.getcCossLeft(size)
      const crossRight = this.getcCossRight(size)
      const win = [...across, ...down, ...[crossRight], ...[crossLeft]]

      this.winningConditions = win
    },
    handleCellPlayed (clickedCell, clickedCellIndex) {
      this.gameState[clickedCellIndex] = this.currentPlayer
      clickedCell.innerHTML = this.currentPlayer

      this.handleLogGame(clickedCellIndex, this.currentPlayer)
    },
    handlePlayerChange () {
      this.currentPlayer = this.getPlayer()
      this.statusDisplay.innerHTML = this.currentPlayerTurn()
    },
    async handleResultValidation () {
      let roundWon = false
      for (let i = 0; i < (this.ROW_SIZE * 2) + 2; i++) {
        const winCondition = this.winningConditions[i]
        const check = []
        for (let i = 0; i < this.ROW_SIZE; i++) {
          check.push(this.gameState[winCondition[i]])
        }
        const winner = check.every((value) => {
          return value === this.currentPlayer
        })
        if (winner) {
          roundWon = true
          console.log(this.logs)
          const response = await this.$axios({
            method: 'post',
            url: 'http://localhost:8000/api/replay',
            data: {
              data: JSON.stringify(this.logs)
            }
          })
          console.log(response)
          this.fetch()
          break
        }
      }

      if (roundWon) {
        this.statusDisplay.innerHTML = this.winningMessage()
        this.gameActive = false
        alert(`${this.currentPlayer} winner !`)
        return
      }

      const roundDraw = !this.gameState.includes('')
      if (roundDraw) {
        this.statusDisplay.innerHTML = this.drawMessage()
        this.gameActive = false
        this.logs.winner = ''
        return
      }

      this.handlePlayerChange()
    },
    handleCellClick (clickedCellEvent) {
      const clickedCell = clickedCellEvent.target

      const clickedCellIndex = parseInt(clickedCell.getAttribute('data-cell-index'))
      if (this.gameState[clickedCellIndex] !== '' || !this.gameActive) {
        return
      }

      this.handleCellPlayed(clickedCell, clickedCellIndex)
      this.handleResultValidation()
    },
    handleRestartGame () {
      this.gameActive = true
      this.currentPlayer = 'X'
      this.gameState = this.getGameState()

      this.statusDisplay.innerHTML = this.currentPlayerTurn()
      const cells = document.querySelectorAll('.cell')
      for (let i = 0; i < cells.length; i++) {
        const cell = cells[i]
        if (cell) {
          cell.innerHTML = ''
        }
      }
      // .forEach(cell => cell.innerHTML)
      delete this.logs.winner
      delete this.logs.log
    },
    handleLogGame (clickedCellIndex, currentPlayer) {
      this.logs.winner = currentPlayer
      this.logs.row = this.ROW_SIZE
      if (this.logs.log == null) { this.logs.log = [] }
      this.logs.log.push({
        player: currentPlayer,
        box: clickedCellIndex
      })
    },
    handleReplayGame () {
      const cells = document.querySelectorAll('.cell')

      for (let i = 0; i < cells.length; i++) {
        const cell = cells[i]
        if (cell) {
          cell.innerHTML = ''
        }
      }

      setTimeout(() => {
        this.logs.log.forEach((log, i) => {
          const cell = cells[log.box]
          setTimeout(() => {
            cell.innerHTML = log.player
            if (i === (this.logs.log.length - 1)) {
              const message = this.logs.winner ? `${this.logs.winner} winner !` : 'draw'
              alert(message)
            }
          }, 600 * i)
        })
      }, 1000)
    },
    start () {
      if (this.startGame) {
        const contentGame = document.querySelector('.game--content')
        this.statusDisplay = document.querySelector('.game--status')
        for (let i = 0; i < this.ROW_SIZE ** 2; i++) {
          const div = document.createElement('div')
          div.setAttribute('data-cell-index', i)
          div.className = 'cell'
          contentGame.appendChild(div)
        }

        contentGame.style.gridTemplateColumns = `repeat(${this.ROW_SIZE}, auto)`

        document.querySelectorAll('.cell').forEach(cell => cell.addEventListener('click', this.handleCellClick))
        document.querySelector('.game--restart').addEventListener('click', this.handleRestartGame)
        document.querySelector('.game--replay').addEventListener('click', this.handleReplayGame)
        this.getGameState()
        this.getWinningConditions(this.ROW_SIZE)
      }
    },
    winningMessage () {
      return `Player ${this.currentPlayer} has won!`
    },
    drawMessage () {
      return 'Game ended in a draw!'
    },
    currentPlayerTurn () {
      return `It's ${this.currentPlayer}'s turn`
    }
  }
}
</script>
<style>
body {
    font-family: "Arial", sans-serif;
}

section {
    text-align: center;
}

.game--content {
    display: grid;
    grid-template-columns: repeat(5, auto);
    width: 300px;
    margin: 0 auto;
    justify-content: center;
}

.cell {
    font-family: "Permanent Marker", cursive;
    width: 100px;
    height: 100px;
    box-shadow: 0 0 0 1px #333333;
    border: 1px solid #333333;
    cursor: pointer;

    line-height: 100px;
    font-size: 60px;
}
</style>
