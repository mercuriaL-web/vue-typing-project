<script>
import { nextTick } from "vue";

export default {
  data() {
    return {
      wordsPool:
        "Vue is an independent, community-driven project. It was created by Evan You in 2014 as a personal side project. Today, Vue is actively maintained by a team of both full-time and volunteer members from all around the world, where Evan serves as the project lead. You can learn more about the story of Vue in this documentary. Vue's development is primarily funded through sponsorships and we have been financially sustainable since 2016. If you or your business benefit from Vue, consider sponsoring us to support Vue's development!".split(" "),
        letters: [],
        timer: 30,
        speedTyping: 0,
        currentIndex: 0,
        record: localStorage.getItem('record'),
        letterStates: [],
        cursorStyle: { left: "0px", top: "0px" }, // Стиль для курсора
    };
  },
  methods: {
    
    setRecord(value) {
      if (value > this.record) {
        this.record = value;
        localStorage.setItem('record', value);
        alert('new record is ' + this.record);
      } 
    },
    calculateSPM() {
      this.letterStates.forEach((letter) => {
        if (letter === "correct") {
          this.speedTyping++;
        }
      });
    },

    addTime() {
      this.timer += 10;
      this.$refs.add.style.color = '#fff'
      setTimeout(() => {
        this.$refs.add.style.color = 'var(--primaryColor)'
      }, 100);

    },

    removeTime() {
      if (this.timer!== 0) {
        this.timer -= 10;
        this.$refs.remove.style.color = '#fff'
      }
      else {
        this.timer = 0;
        this.$refs.remove.style.color = '#f55'
      }
      setTimeout(() => {
        this.$refs.remove.style.color = 'var(--primaryColor)'
      }, 100);

    },
    reset() {
      this.letters = [];
      this.currentIndex = 0;
      this.letterStates = [];
      this.$refs.words.style.marginTop = 0;
      this.cursorStyle = { left: "0px", top: "0px" };
    },
    randomWord() {
      const wordsCount = this.wordsPool.length;
      const randomIndex = Math.floor(Math.random() * wordsCount);
      return this.wordsPool[randomIndex];
    },
    newGame() {
      this.reset();
      this.speedTyping = 0;

      // Генерация случайных слов
      for (let i = 0; i < 100; i++) {
        const word = this.randomWord();
        this.letters.push(...word.split(""), " ");
      }

      // Заполняем состояние букв
      this.letterStates = new Array(this.letters.length).fill(null);

      // Обновляем курсор после рендеринга DOM
      nextTick(() => {
        this.updateCursorPosition();
      });

      // Запускаем таймер
      const intervalId = setInterval(() => {
        this.timer--;
        if (this.timer === 0) {
          clearInterval(intervalId);
          this.calculateSPM();
          this.timer = 30;
          this.setRecord(this.speedTyping);
          this.reset();
        }
      }, 1000);
    },
    pressingKey(event) {
      const key = event.key;
      const expectedKey = this.letters[this.currentIndex];

      if (key.length === 1) {
        this.letterStates[this.currentIndex] =
          key === expectedKey ? "correct" : "incorrect";
        this.letterStates[this.currentIndex + 1] = "current";
        this.currentIndex++;
      } else if (key === "Backspace" && this.currentIndex > 0) {
        this.letterStates[this.currentIndex - 1] = "default";
        this.currentIndex--;
      }

      if (
        this.$refs.letter[this.currentIndex].getBoundingClientRect().top > 250
      ) {
        console.log(this.$refs.words.style);
        this.$refs.words.style.marginTop =
          parseInt(this.$refs.words.style.marginTop || "0px") - 35 + "px";
      }

      // Обновляем курсор после изменения индекса
      nextTick(() => {
        this.updateCursorPosition();
      });
    },
    updateCursorPosition() {
      if (this.$refs.letter && this.$refs.letter[this.currentIndex]) {
        const nextLetter = this.$refs.letter[this.currentIndex];
        const rect = nextLetter.getBoundingClientRect();

        // Обновляем позицию курсора
        this.cursorStyle = {
          left: rect.left + "px",
          top: rect.top + "px",
        };
      }
    },
  },
};
</script>

<template>
  <main>
    <h1>Speed Typing Test</h1>
    <div id="header">
      <div id="info">
        <span
          ref="remove"
          v-show="speedTyping === 0"
          style="cursor: pointer"
          @click="this.removeTime()"
          >- </span
        >{{ speedTyping === 0 ? timer : "SPM is " + speedTyping
        }}<span
          ref="add"
          v-show="speedTyping === 0"
          style="cursor: pointer"
          @click="this.addTime()"
        >
          +</span
        >
        {{ 'Record: ' + record }}
      </div>
      <div id="buttons">
        <button @click="newGame()">New game</button>
      </div>
    </div>
    <div id="game" tabindex="0" @keydown="pressingKey($event)" ref="game">
      <div id="words" ref="words">
        <span
          v-for="(letter, index) in letters"
          :key="index"
          ref="letter"
          :class="{
            current: index === currentIndex,
            correct: letterStates[index] === 'correct',
            incorrect: letterStates[index] === 'incorrect',
          }"
        >
          {{ letter }}
        </span>
      </div>
      <!-- Привязываем стили курсора -->
      <div
        id="cursor"
        :style="cursorStyle"
        v-show="this.letters.length > 0"
      ></div>
    </div>
  </main>
</template>

<style scoped>
main {
  width: 600px;
  margin: 50px auto;
}

#header {
  display: grid;
  grid-template-columns: 1fr 1fr;
  margin: 20px 6px 30px;
}

#buttons {
  text-align: right;
}

#info {
  color: var(--primaryColor);
  user-select: none;
}

span {
  user-select: none;
}


button {
  background: var(--primaryColor);
  border: 0;
  color: var(--bgColor);
  padding: 5px 20px;
  border-radius: 10px;
  cursor: pointer;
}

#words {
  display: inline-block;
  margin: 0 5px;
  color: var(--textSecondary);
}

#game {
  line-height: 35px;
  height: 105px;
  overflow: hidden;
  position: relative;
}

#game:focus {
  outline: 0;
}

#focus-error {
  position: absolute;
  inset: 0;
  text-align: center;
  padding-top: 35px;
}

.default {
  color: var(--textSecondary);
}

.correct {
  color: #fff;
}

.incorrect {
  color: #f55;
}

.current {
  font-weight: bold;
  color: var(--primaryColor);
}

@keyframes blink {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

#cursor {
  width: 2px;
  height: 1.6rem;
  background: var(--primaryColor);
  position: fixed;
  top: 195px;
  animation: blink 1s infinite;
}

.move__cursor {
  left: 2px;
}
</style>
