<script>
import { nextTick } from 'vue';

export default {
  data() {
    return {
      wordsPool: 'Vue is an independent, community-driven project. It was created by Evan You in 2014 as a personal side project. Today, Vue is actively maintained by a team of both full-time and volunteer members from all around the world, where Evan serves as the project lead. You can learn more about the story of Vue in this documentary. Vue\'s development is primarily funded through sponsorships and we have been financially sustainable since 2016. If you or your business benefit from Vue, consider sponsoring us to support Vue\'s development!'.split(' '),
      letters: [],
      currentIndex: 0,
      letterStates: [],
      cursorStyle: { left: '0px', top: '0px' } // Стиль для курсора
    };
  },
  methods: {
    randomWord() {
      const wordsCount = this.wordsPool.length;
      const randomIndex = Math.floor(Math.random() * wordsCount);
      return this.wordsPool[randomIndex];
    },
    newGame() {
      this.letters = [];
      this.currentIndex = 0;
      this.letterStates = [];

      // Генерация случайных слов
      for (let i = 0; i < 100; i++) {
        const word = this.randomWord();
        this.letters.push(...word.split(''), ' ');
      }

      // Заполняем состояние букв
      this.letterStates = new Array(this.letters.length).fill(null);

      // Обновляем курсор после рендеринга DOM
      nextTick(() => {
        this.updateCursorPosition();
      });
    },
    pressingKey(event) {
      const key = event.key;
      const expectedKey = this.letters[this.currentIndex];

      if (key.length === 1) {
        this.letterStates[this.currentIndex] = key === expectedKey ? 'correct' : 'incorrect';
        this.currentIndex++;
      } else if (key === 'Backspace' && this.currentIndex > 0) {
        this.letterStates[this.currentIndex-1] = 'default';
        this.currentIndex--;
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
          left: rect.left + 'px',
          top: rect.top + 'px'
        };
      }
    }
  }
};
</script>

<template>
  <main>
    <h1>Speed Typing Test</h1>
    <div id="header">
      <div id="info">30</div>
      <div id="buttons">
        <button @click="newGame()">New game</button>
      </div>
    </div>
    <div id="game" tabindex="0" @keydown="pressingKey($event)">
      <div id="words">
        <span v-for="(letter, index) in letters" 
              :key="index" 
              ref="letter"  
              :class="{current: index === currentIndex, correct: letterStates[index] === 'correct', incorrect: letterStates[index] === 'incorrect'}">
          {{ letter }}
        </span>
      </div>
      <!-- Привязываем стили курсора -->
      <div id="cursor" :style="cursorStyle" v-show="this.letters.length > 0"></div>
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
  
  .letter.current {
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