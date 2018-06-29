<template>
  <div>

<!--LOGIN FORM-->
   
    <div class="login" v-show="showLogin">
      <div class="login__form">
        <label class="login__label" for="name" style="color: #111;">Логин</label>
        <input class="login__input" type="text" id="name" required placeholder="Введите имя" v-model="name">
        <button class="login__button" @click="logIn">Войти</button>
      </div>
    </div>


    <!--GAME-->
    <div v-show="!showLogin && showGrid">
      <!-- <div class="scoreBoard">
        <span>O has {{ wins.O }} wins</span>
        <h2>Score Board</h2>
        <span>X has {{ wins.X }} wins</span>
      </div> -->
      <div id="app">
        <div id="details">
          <h1>Крестики-нолики</h1>
          <!-- <h2>Матч #{{ matches + 1 }}</h2> -->
        </div>
        <grid ref="gameGrid"></grid>
        <button class="restart">Осталось: <span id="timer_inp">60</span> сек.</button>
      </div>
    </div>

     <!--PRELOADER-->
  <div class="preloader" v-show="!showLogin && !showGrid">
    <h2 class="title">Ожидаем соперника..</h2>
    <div class="container">
      <div class="item-1"><div></div></div>
      <div class="item-2"><div></div></div>
      <div class="item-3"><div></div></div>
      <div class="item-4"><div></div></div>
      <div class="item-5"><div></div></div>
      <div class="item-6"><div></div></div>
      <div class="item-7"><div></div></div>
      <div class="item-8"><div></div></div>
      <div class="item-9"><div></div></div>
    </div>
  </div>

  </div>
</template>

<script>
import Grid from './components/Grid.vue';
import axios from 'axios';

export default {
  components: { Grid },
  name: 'app',
  data() {
    return {
      name: '',
      showLogin: false,
      showGrid: false,
      checkFlag: true,
      matches: 0,
      wins: {
        O: 0,
        X: 0
      }
    };
  },

  methods: {
    timer: function() {
      let obj = document.getElementById('timer_inp');
      obj.innerHTML--;
      if (obj.innerHTML == 0) {
        this.$refs.gameGrid.randomTurn();
        setTimeout(function() {
          alert('Время вышло');
        }, 300);
      } else {
        if (this.$refs.gameGrid.active) {
          setTimeout(this.timer, 1000);
        } else {
          obj.innerHTML = 60;
        }
      }
    },
    logIn: function() {
      axios
        .post('/api/games/player_login', {
          name: this.name
        })
        .then(response => {
          console.log(response);
          if (response.data.status === true) {
            this.checkFlag = true;
          } else {
            alert('Something wrong');
          }
        })
        .catch(err => {
          console.log('error');
        });
    },
    checkState: function() {
      if (this.checkFlag) {
        axios
          .get('/api/games/current')
          .then(response => {
            if (response.data === null || response.data.status === null) {
              this.showLogin = true;
              this.checkFlag = false;
            } else if (Object.keys(response.data.players).length < 2) {
              this.showGrid = false;
              this.showLogin = false;
            } else {
              if (response.data.active === false) {
                console.log(this.$refs.gameGrid);
                this.checkFlag = true;
                this.$refs.gameGrid.setDisabled();
              } else {
                this.checkFlag = false;
                for (let i in response.data.grid) {
                  if (response.data.grid[i] === 0) {
                    this.$refs.gameGrid.cells[i] = 'o';
                  } else if (response.data.grid[i] === 1) {
                    this.$refs.gameGrid.cells[i] = 'x';
                  } else {
                    this.$refs.gameGrid.cells[i] = '';
                  }
                }
                setTimeout(this.timer, 1000);
                this.$refs.gameGrid.setEnabled();
              }
              if (response.data.result === 0) {
                this.showGrid = true;
                if (response.data.mark === 1) {
                  this.$refs.gameGrid.mark = 'x';
                } else if (response.data.mark === 0) {
                  this.$refs.gameGrid.mark = 'o';
                }
              } else if (response.data.result === 1) {
                this.checkFlag = false;
                alert('WIN!');
              } else if (response.data.result === 2) {
                this.checkFlag = false;
                alert('LOOOSER haha');
              } else if (response.data.result === -1) {
                this.checkFlag = false;
                alert('Draw');
              }
            }
          })
          .catch(err => {
            console.log('error');
          });
      }
      setTimeout(this.checkState, 2000);
      clearTimeout(this.timer);
    }
  },

  watch: {},

  created() {
    Event.$on('disableCheck', () => {
      this.checkFlag = false;
    });
    Event.$on('enableCheck', () => {
      this.checkFlag = true;
    });
  },

  beforeMount() {
    this.checkFlag = true;
    this.checkState();
  }
};
</script>

<style>
body {
  background-color: #fff;
  color: #111;
  font-family: 'Dosis', Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin: 0px;
}

#app {
  margin: 0 auto;
  max-width: 270px;
  color: #34495e;
}

h1 {
  text-transform: uppercase;
  font-weight: bold;
  font-size: 1.5em;
}

.timer {
  text-transform: uppercase;
  font-weight: bold;
  font-size: 1em;
  margin-top: 10px;
}

.restart {
  background-color: #e74c3c;
  color: #fff;
  border: 0px;
  border-bottom-left-radius: 10px;
  border-bottom-right-radius: 10px;
  font-family: 'Dosis', Helvetica, sans-serif;
  font-size: 1.4em;
  font-weight: bold;
  margin: 0px;
  padding: 15px;
  width: 100%;
}

.restart:hover {
  background-color: #c0392b;
  cursor: pointer;
}

.scoreBoard {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
  width: 100%;
  height: 15px;
  background-color: #df4751;
  box-shadow: 10px solid #fff;
  padding: 20px;
  overflow-x: none;
  text-transform: uppercase;
}

.scoreBoard h2 {
  margin: 0px;
}

.scoreBoard span {
  float: right;
  font-size: 1.5em;
  font-weight: bold;
  margin-left: 20px;
}

.login {
  position: relative;
}

.login__form {
  position: absolute;
  top: 40vh;
  left: 50%;
  transform: translate(-50%, -50%);
  border: 3px solid #3d9c55;
  padding: 25px 30px;
  border-radius: 10px;
}

.login__button {
  display: block;
  text-align: center;
  margin: 20px auto 0 auto;
  width: 190px;
  padding: 5px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  background-color: #3d9c55;
  color: #fff;
  text-transform: uppercase;
  transition: all 0.3s;
}

.login__button:hover {
  opacity: 0.9;
}

.login__button:active {
  opacity: 0.7;
}

.login__label {
  display: block;
  margin-bottom: 20px;
}

.login__input {
  padding: 5px;
  border-radius: 3px;
  border: 1px solid #837f7f;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 200px;
  overflow: hidden;
}

.item-1 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-1 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 100ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-1 {
  10% {
    transform: rotate(250deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(250deg) translateX(-100px);
  }
  100% {
    transform: rotate(250deg) translate(0, 0px);
  }
}
.item-1 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #3d9c55;
}

.item-2 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-2 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 200ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-2 {
  10% {
    transform: rotate(375deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(375deg) translateX(-100px);
  }
  100% {
    transform: rotate(375deg) translate(0, 0px);
  }
}
.item-2 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #3d9c55;
}

.item-3 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-3 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 300ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-3 {
  10% {
    transform: rotate(500deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(500deg) translateX(-100px);
  }
  100% {
    transform: rotate(500deg) translate(0, 0px);
  }
}
.item-3 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #3d9c55;
}

.item-4 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-4 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 400ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-4 {
  10% {
    transform: rotate(625deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(625deg) translateX(-100px);
  }
  100% {
    transform: rotate(625deg) translate(0, 0px);
  }
}
.item-4 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #df4751;
}

.item-5 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-5 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 500ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-5 {
  10% {
    transform: rotate(750deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(750deg) translateX(-100px);
  }
  100% {
    transform: rotate(750deg) translate(0, 0px);
  }
}
.item-5 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #df4751;
}

.item-6 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-6 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 600ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-6 {
  10% {
    transform: rotate(875deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(875deg) translateX(-100px);
  }
  100% {
    transform: rotate(875deg) translate(0, 0px);
  }
}
.item-6 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #df4751;
}

.item-7 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-7 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 700ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-7 {
  10% {
    transform: rotate(1000deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(1000deg) translateX(-100px);
  }
  100% {
    transform: rotate(1000deg) translate(0, 0px);
  }
}
.item-7 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #0ff140;
}

.item-8 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-8 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 800ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-8 {
  10% {
    transform: rotate(1125deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(1125deg) translateX(-100px);
  }
  100% {
    transform: rotate(1125deg) translate(0, 0px);
  }
}
.item-8 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #0ff140;
}

.item-9 {
  width: 100px;
  height: 100px;
  position: absolute;
  animation: right-9 4s infinite cubic-bezier(0, 0, 0.49, 1.02);
  animation-delay: 900ms;
  display: flex;
  justify-content: center;
  align-items: center;
}
@keyframes right-9 {
  10% {
    transform: rotate(1250deg) translate(0, 0px);
  }
  50%,
  65% {
    transform: rotate(1250deg) translateX(-100px);
  }
  100% {
    transform: rotate(1250deg) translate(0, 0px);
  }
}
.item-9 > * {
  width: 20px;
  height: 20px;
  background: #f583a1;
  border-radius: 50%;
  background-color: #0ff140;
}

.title {
  color: #3d9c55;
  margin-top: 20px;
}
</style>
