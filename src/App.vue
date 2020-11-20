<template>
  <div class="wrapper">
    <h1>Simon Says</h1>
    <div class="game-board">
      <div class="simon">
        <ul ref="simon-conteiner">
          <li
            :class="[{ active: active === 1 }, 'tile', 'red']"
            data-tile="1"
          ></li>
          <li
            :class="[{ active: active === 2 }, 'tile', 'blue']"
            data-tile="2"
          ></li>
          <li
            :class="[{ active: active === 3 }, 'tile', 'yellow']"
            data-tile="3"
          ></li>
          <li
            :class="[{ active: active === 4 }, 'tile', 'green']"
            data-tile="4"
          ></li>
        </ul>
      </div>
    </div>
    <div class="game-info">
      <h2>
        Round: <span>{{ round }}</span>
      </h2>
      <button @click="startGame()" type="button">Start</button>
      <p v-show="showLose">
        Sorry, you lost after <span>{{ showLose }}</span> rounds!
      </p>
    </div>
    <div class="game-options">
      <h2>Game Options:</h2>
      <v-radio-group
        :map="modeMap"
        :active="mode"
        @input="mode = $event"
        name="mode"
      />
      <h2>Lvl:</h2>
      <v-radio-group
        :map="LVLMap"
        :active="lvl"
        @input="lvl = $event"
        name="lvl"
      />
    </div>
  </div>
</template>

<script>
import VRadioGroup from './components/v-radio-group.vue';
import audioPath from './audioPath';

export default {
  name: 'App',

  components: { VRadioGroup },

  data() {
    return {
      sequence: [],
      mode: 'normal',
      round: 0,
      active: null,
      countAnsver: 0,
      showLose: 0,
      lvl: 1500,
      modeMap: {
        normal: 'Normal',
        'sound-only': 'Sound Only',
        'light-only': 'Light Only',
        'free-board': 'Free-Board'
      },
      LVLMap: {
        1500: 'Easy',
        1000: 'Normal',
        500: 'Hard'
      }
    };
  },

  methods: {
    startGame() {
      this.sequence = [];
      this.round = 0;
      this.showLose = 0;
      this.nextRound();
    },
    async nextRound() {
      this.sequence.push(this.randomNumber());
      this.round++;
      await this.animate();
      this.countAnsver = 0;
      this.addEvent();
    },

    async animate() {
      for (let i = 0; i < this.sequence.length; i++) {
        await this.setActive(this.sequence[i]);
      }
    },

    setActive(i) {
      return new Promise(resolve => {
        window.setTimeout(() => {
          resolve();
        }, 50);
      }).then(() => {
        this.playAudio(i);
        return this.activeLight(i);
      });
    },

    async setAnsver(event) {
      await this.setActive(+event.target.dataset.tile);
      if (this.sequence[this.countAnsver] === +event.target.dataset.tile) {
        this.countAnsver++;
        if (this.sequence.length === this.countAnsver) {
          this.removeEvent();
          this.nextRound();
          return;
        }
      } else {
        this.endGame();
      }
    },

    endGame() {
      this.removeEvent();
      this.sequence = [];
      this.countAnsver = 0;
      this.showLose = this.round;
      this.round = 0;
    },

    addEvent() {
      this.$refs['simon-conteiner'].addEventListener('click', this.setAnsver);
    },

    removeEvent() {
      this.$refs['simon-conteiner'].removeEventListener(
        'click',
        this.setAnsver
      );
    },

    randomNumber() {
      return Math.floor(Math.random() * 4 + 1);
    },

    activeLight(i) {
      if (this.mode !== 'sound-only') {
        return new Promise(resolve => {
          this.active = i;
          window.setTimeout(() => {
            this.active = null;
            resolve();
          }, +this.lvl);
        });
      }
    },

    playAudio(i) {
      if (this.mode !== 'light-only') {
        let audio = new Audio(audioPath[i - 1]);
        audio.play();
      }
    }
  }
};
</script>

<style lang="sass">
body
  font-family: Arial, serif
  color: #333
  -webkit-user-select: none

  /* Chrome/Safari
  -moz-user-select: none

  /* Firefox
  -ms-user-select: none

  /* IE10+
  -o-user-select: none
  user-select: none

h1
  margin: 1em 0 2em
  text-align: center

ul
  list-style: none
  padding: 0
  margin: 0

li
  padding: 0
  margin: 0

p[data-action="lose"]
  display: none

.active
  opacity: 1 !important

.clearfix
  width: 100%
  clear: both

.wrapper
  width: 540px
  margin: 0 auto

.container
  width: 305px

.simon
  background: #fff
  position: relative
  float: left
  margin-right: 3em
  width: 302px
  height: 295px
  -webkit-border-radius: 150px 150px 150px 150px
  border-radius: 150px 150px 150px 150px
  -moz-box-shadow: 2px 1px 12px #aaa
  -webkit-box-shadow: 2px 1px 12px #aaa
  -o-box-shadow: 2px 1px 12px #aaa
  box-shadow: 2px 1px 12px #aaa

.tile
  opacity: 0.6
  -webkit-transition: opacity 250ms ease
  -moz-transition: opacity 250ms ease
  -ms-transition: opacity 250ms ease
  -o-transition: opacity 250ms ease
  transition: opacity 250ms ease

  &.lit
    opacity: 1

.red, .blue, .yellow, .green
  height: 290px
  -webkit-border-radius: 150px 150px 150px 150px
  border-radius: 150px 150px 150px 150px
  position: absolute
  text-indent: 10000px

.red:hover, .blue:hover, .yellow:hover, .green:hover
  border: 2px solid black

.red
  background: #FF5643
  clip: rect(0px, 300px, 150px, 150px)
  width: 296px

.blue
  background: dodgerblue
  clip: rect(0px, 150px, 150px, 0px)
  width: 300px

.yellow
  background: #FEEF33
  clip: rect(150px, 150px, 300px, 0px)
  width: 300px

.green
  background: #BEDE15
  clip: rect(150px, 300px, 300px, 150px)
  width: 296px

.game-info
  margin-top: 90px

  button
    width: 5em
    box-sizing: border-box
    font-size: 1.4em
    -webkit-border-radius: 10px 10px 10px 10px
    border-radius: 10px 10px 10px 10px
    background: #6DABE8
    border: none
    padding: 0.3em 0.6em

    &:hover
      background: #78BCFF

.game-options
  float: right
  h2
    margin-top: 30px
    margin-bottom: 0

  input[type="radio"]
    margin-right: 10px

.hoverable:hover
  cursor: pointer
</style>
