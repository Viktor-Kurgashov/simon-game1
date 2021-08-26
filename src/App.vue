<template>
  <div class="circle">
    <div id="cell-1" @mousedown="cellMouseDown" @mouseup="cellMouseUp" @click="observe" />
    <audio>  <source src="./assets/1.mp3" type="audio/mpeg">  </audio>

    <div id="cell-2" @mousedown="cellMouseDown" @mouseup="cellMouseUp" @click="observe" />
    <audio>  <source src="./assets/2.mp3" type="audio/mpeg">  </audio>

    <div id="cell-3" @mousedown="cellMouseDown" @mouseup="cellMouseUp" @click="observe" />
    <audio>  <source src="./assets/4.mp3" type="audio/mpeg">  </audio>

    <div id="cell-4" @mousedown="cellMouseDown" @mouseup="cellMouseUp" @click="observe" />
    <audio>  <source src="./assets/3.mp3" type="audio/mpeg">  </audio>
  </div>

  <div class="difficulty-levels">
    <label for="level-easy">
      <input type="radio" name="d" value="easy" id="level-easy" v-model="difficultyLevel"> Easy
    </label>

    <label for="level-medium">
      <input type="radio" name="d" value="medium" id="level-medium" v-model="difficultyLevel" selected> Medium
    </label>

    <label for="level-hard">
      <input type="radio" name="d" value="hard" id="level-hard" v-model="difficultyLevel"> Hard
    </label>
  </div>

  <span id="notification-display">{{ notificationText }}</span>

  <button @click="startGame" class="start-button">Start again</button>
</template>

<script>
export default {
  name: 'App',

  data() {
    return {
      game: [], // последовательность нажатия кнопок
      difficultyLevel: 'hard', // 'easy' || 'medium' || 'hard'

      levelLength: 1, // текущий уровень и количество нажатий кнопок на нём
      maxLevel: 5,

      levelPushedButtons: 0, // нажатых правильных кнопок подряд
      notification: '' // '' || 'Start the game' || 'Game over' || 'WIN!!!' - Текст уведомления над кнопкой старта
    }
  },

  computed: {
    interval() {  // интервал между нажатиями при воспроизведении последовательности кнопок
      return (this.difficultyLevel === 'hard') ? 400 : (this.difficultyLevel == 'medium') ? 1000 : 1500
    },
    notificationText() {
      return (this.notification === '') ? `${this.levelLength}/${this.maxLevel}` : this.notification
    }
  },

  methods: {
    startGame() {
      this.game = []
      while (this.game.length < this.maxLevel) {  // создаёт последовательность ячеек
        this.game.push(Math.floor(Math.random() * 4) + 1)
      }
      this.notificationControl('clean')
      this.playback(1) // показать первую ячейку
    },


    observe(event) {  // запускается при нажатии на ячейку, следит за ходами игрока
      if (this.game.length === 0) {  // уведомление если игра не начата
        this.notificationControl('click before start')
      }

      else if (event.target.id[5] == this.game[this.levelPushedButtons]) {
        this.levelPushedButtons += 1  // нажата правильная ячейка, но не последняя на текущем уровне

        if (this.levelPushedButtons === this.levelLength) { // нажата последняя кнопка на текущем уровне
          this.levelPushedButtons = 0

          if (this.levelLength < this.maxLevel) {  // уровень пройден
            this.notificationControl('next level')
            this.levelLength += 1  // переход на следующий
            setTimeout(() => this.playback(this.levelLength), 1000)  // показывает последовательность на следующем уровне
          }          
          else if (this.levelLength === this.maxLevel) { // пройден последний уровень, победа
            this.notificationControl('win')
            this.levelLength = 1
            this.game = []
          }
        }
      }

      else { // нажата неправильная ячейка, сброс всего
        this.notificationControl('game over')
        this.levelPushedButtons = 0
        this.levelLength = 1
        this.game = []
      }
    },


    playback(number) { // воспроизводит последовательность ячеек на текущем уровне
      let counter = 0;

      const func = () => {
        this.highlightCell('cell-' + this.game[counter], this.interval)
        counter += 1;
        if (counter < number) setTimeout(func, this.interval)
      }
      func()
    },

    highlightCell(cell, interval) {  // кусок кода из функции playback
      cell = document.getElementById(cell)
      cell.classList.add('active-cell');

      if (interval === 400) {
        cell.nextElementSibling.currentTime = 0;
        cell.nextElementSibling.play()            
      } else {
        cell.nextElementSibling.play()
      }
      setTimeout(() => cell.classList.remove('active-cell'), interval * 0.9)
    },


    notificationControl(act) {
      const notification = document.getElementById('notification-display')

      if (act === 'clean') {
        this.notification = ''
        notification.style.background = ''
      }
      else if (act === 'next level') {  // моргнуть зелёным
        notification.style.backgroundColor = 'mediumseagreen'
        setTimeout(() => notification.style.backgroundColor = '', 200)
      }
      else if (act === 'win') {
        notification.style.backgroundColor = 'mediumseagreen'
        this.notification = 'WIN!!!'
      }
      else if (act === 'click before start') {
        notification.style.backgroundColor = 'tomato'
        this.notification = 'Start the game'
      }
      else if (act === 'game over') {
        notification.style.backgroundColor = 'tomato'
        this.notification = 'Game over'
      }
    },


    cellMouseDown(event) {
      event.target.classList.add('active-cell')
      event.target.nextElementSibling.play()
    },
    cellMouseUp(event) {
      event.target.classList.remove('active-cell')
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}
body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  width: 600px;
  margin: 100px auto 0;
}
#app:after {
  content: "";
  clear: both;
  display: table;
}

.circle {
  position: relative;
  float: left;
  width: 300px;
  height: 300px; 
}
#app > .circle > .active-cell {
  border-color: crimson;
  opacity: 0.5;
}


#cell-1, #cell-2, #cell-3, #cell-4 {
  cursor: pointer;
  position: absolute;
  width: 50%;
  height: 50%;
}
#cell-1 {
  border-top: 5px solid white;
  border-left: 5px solid white;

  border-radius: 100% 0 0 0;
  background: dodgerblue;
  bottom: 50%;
  right: 50%;
}
#cell-2 {
  border-top: 5px solid white;
  border-right: 5px solid white;

  border-radius: 0 100% 0 0;
  background: tomato;
  bottom: 50%;
  left: 50%;
}
#cell-3 {
  border-bottom: 5px solid white;
  border-left: 5px solid white;

  border-radius: 0 0 0 100%;
  background: rgb(255, 228, 77);
  top: 50%;
  right: 50%;
}
#cell-4 {
  border-bottom: 5px solid white;
  border-right: 5px solid white;

  border-radius: 0 0 100% 0;
  background: rgb(101, 231, 101);
  top: 50%;
  left: 50%;
}


.difficulty-levels {
  float: right;
  width: 250px;
  font-size: 1.25em;
  margin-top: 35px;
}
.difficulty-levels > label {
  display: block;
  margin-bottom: 7px;
}


#notification-display {
  float: right;
  display: block;
  width: 250px;
  height: 60px;
  margin-top: 15px;

  text-align: center;
  line-height: 60px;  
  font-size: 2em;

  background-color: whitesmoke;
  transition: background-color 0.1s;
  border-radius: 5px;
}

.start-button {
  display: block;
  float: right;
  font-size: 1.25em;
  width: 250px;
  padding: 7px 0;
  margin-top: 17px;
  cursor: pointer;
}
</style>
