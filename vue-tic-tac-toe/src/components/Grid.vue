<template>
	<div>
		<div class="gameStatus" :class="gameStatusColor">
			Ходите уже!
		</div>
		<table class="grid">
		  <tr v-for="i in 3">
		    <cell 
        :disabled="!active" 
        :mark="cells[(i - 1)*3 + j - 1]" 
        :name="((i - 1)*3 + j - 1).toString()" 
        v-for="j in 3"></cell>
		  </tr>
		</table>
	</div>
</template>

<script>
import axios from 'axios';
import Cell from './Cell.vue';

export default {
  components: { Cell },

  data() {
    return {
      // can be O or X
      //activePlayer: 'O',
      // maintains the status of the game: turn or win or draw
      gameStatus: 'turn',
      active: false,
      mark: '',
      gameStatusMessage: `O's turn`,
      // status color is used as background color in the status bar
      // it can hold the name of either of the following CSS classes
      // statusTurn (default) is yellow for a turn
      // statusWin is green for a win
      // statusDraw is purple for a draw
      gameStatusColor: 'statusTurn',
      // no. of moves played by both players in a single game (max = 9)
      /* moves: 0, */
      // stores the placement of X and O in cells by their cell number
      /* cells: {
        0: '',
        1: '',
        2: '',
        3: '',
        4: '',
        5: '',
        6: '',
        7: '',
        8: ''
      } */
      cells: [null, null, null, null, null, null, null, null, null]
      // contains all (8) possible winning conditions
      /* winConditions: [
        [0, 1, 2],
        [3, 4, 5],
        [6, 7, 8], // rows
        [0, 3, 6],
        [1, 4, 7],
        [2, 5, 8], // columns
        [0, 4, 8],
        [2, 4, 6] // diagonals
      ] */
    };
  },

  computed: {
    // helper property to get the non-active player
    /*  nonActivePlayer() {
      if (this.activePlayer === 'O') {
        return 'X';
      }

      return 'O';
    } */
  },

  /*  watch: {
    // watches for change in the value of gameStatus and changes the status
    // message and color accordingly
    gameStatus() {
      if (this.gameStatus === 'win') {
        this.gameStatusColor = 'statusWin';

        return;
      } else if (this.gameStatus === 'draw') {
        this.gameStatusColor = 'statusDraw';

        this.gameStatusMessage = 'Draw !';

        return;
      }

      this.gameStatusMessage = `${this.activePlayer}'s turn`;
    }
  }, */

  methods: {
    turn: function(cellNumber) {
      this.cells[cellNumber] = this.mark;
      this.active = false;
      // SEND [turn] params: {index: cellNumber}
      axios
        .post('/api/games/current/turn', { index: cellNumber })
        .then(response => {
          console.log(response);
        })
        .catch(err => {
          console.log('error');
        });
      Event.$emit('enableCheck');
    },
    randomTurn: function() {
      let a = parseInt(Math.random() * 1000) % 9;
      console.log(a);
      while (this.cells[a] != '') {
        a = parseInt(Math.random() * 1000) % 9;
      }
      this.turn(a);
    },
    setDisabled: function() {
      this.active = false;
    },
    setEnabled: function() {
      this.active = true;
    }
  },

  created() {
    // listens for a strike made by the user on cell
    // it is called by the Cell component
    Event.$on('strike', cellNumber => {
      this.turn(cellNumber);
    });

    // listens for a restart button press
    // the data of the component is reinitialized
    // it is called by the App component
    Event.$on('gridReset', () => {
      Object.assign(this.$data, this.$options.data());
    });
  }
};
</script>

<style>
.grid {
  background-color: #34495e;
  color: #fff;
  width: 100%;
  border-collapse: collapse;
}

.gameStatus {
  margin: 0px;
  padding: 15px;
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  background-color: #0ff140;
  color: #fff;
  font-size: 1.4em;
  font-weight: bold;
}

.statusTurn {
  background-color: #3d9c55;
}

.statusWin {
  background-color: #2ecc71;
}

.statusDraw {
  background-color: #9b59b6;
}
</style>
