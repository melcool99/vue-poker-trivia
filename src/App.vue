<script setup lang="ts">
import { ref, watch } from 'vue'
import PokerTable from './components/PokerTable.vue'
import StartModal from './components/StartModal.vue'
import * as PokerSolver from 'pokersolver'

const playerCards = ref<any>([])

const communityCards = ref<any>([])

const suits = ['H', 'D', 'C', 'S']
const ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A']
const deck = ref<any>([])

const handValues = ref([
  'High Card',
  'Pair',
  'Two Pair',
  'Three of a Kind',
  'Straight',
  'Flush',
  'Full House',
  'Four of a Kind',
  'Straight Flush',
  'Royal Flush'
])

const VALUE = 'Royal Flush'
const points = ref(10)
const isAnswered = ref(false)
const timerSeconds = ref(10)
const intervalStep = ref(1000)
const answerType = ref<null | boolean>(null)
const showModal = ref(true)
let startTime = ref<any | number>(null)
let totalTime = ref<null | number>(null)
let intervalId: any = null

const createDeck = () => {
  for (const suit of suits) {
    for (const rank of ranks) {
      deck.value.push({ suit, rank })
    }
  }
}

const shuffleDeck = () => {
  for (let i = deck.value.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[deck.value[i], deck.value[j]] = [deck.value[j], deck.value[i]]
  }
}

const dealCards = () => {
  playerCards.value = deck.value.splice(0, 2)
  deck.value.splice(0, 1)
  communityCards.value = deck.value.splice(0, 3)
  deck.value.splice(0, 1)
  communityCards.value.push(...deck.value.splice(0, 1))
  deck.value.splice(0, 1)
  communityCards.value.push(...deck.value.splice(0, 1))
}

const getBestHand = () => {
  const allCards = [...playerCards.value, ...communityCards.value]
  const hand = allCards.map((card) => `${card.rank}${card.suit[0].toLowerCase()}`)
  // const result = PokerSolver.Hand.solve(hand)
}

const handleAnswer = (value: String) => {
  isAnswered.value = true
  clearInterval(intervalId)

  if (value && value === VALUE) {
    answerType.value = true
    points.value += 1
  } else {
    answerType.value = false
    points.value -= points.value >= 3 ? 3 : points.value
  }
  intervalStep.value = intervalStep.value - 10
  setTimeout(restartRound, 1000)
}

const startGame = () => {
  showModal.value = false
  startTime.value = Date.now()
  points.value = 10
  timerSeconds.value = 10
  createDeck()
  startRound()
}

const checkPoints = () => {
  if (points.value <= 0) {
    totalTime.value = Date.now() - startTime.value
    let timeInSeconds = totalTime.value / 1000
    let minutes = Math.floor(timeInSeconds / 60)
    let seconds = Math.floor(timeInSeconds % 60)
    points.value = 0
    timerSeconds.value = 0
    alert(`Time spent: ${minutes} minutes and ${seconds} seconds`)
    showModal.value = true
    clearInterval(intervalId)
  }
}

watch(points, checkPoints)

const startRound = () => {
  isAnswered.value = false

  shuffleDeck()
  dealCards()
  getBestHand()
  intervalId = setInterval(() => {
    if (timerSeconds.value > 0) {
      timerSeconds.value--
    } else {
      points.value -= points.value >= 3 ? 3 : points.value

      clearInterval(intervalId)
      setTimeout(restartRound, 1000)
    }
    intervalStep.value = intervalStep.value - 10
  }, intervalStep.value)
}

const restartRound = () => {
  if (points.value > 0) {
    clearInterval(intervalId)
    timerSeconds.value = 10
    intervalStep.value = 1000
    isAnswered.value = false
    startRound()
  }
}
</script>

<template>
  <StartModal v-model="showModal" v-if="showModal">
    <h2 class="">Start a new game?</h2>
    <button class="btn" @click="startGame">Start</button>
  </StartModal>
  <main class="container" :class="!timerSeconds && 'incorrect-answer'" v-else>
    <section class="info-data">
      <p class="points">Stamina: {{ points }}</p>
      <p class="timer">Timer: {{ timerSeconds }}</p>
    </section>
    <section class="hand-values-container">
      <h2>Guess your hand!</h2>
      <div class="hand-values">
        <button
          class="btn"
          :class="{
            'correct-answer': isAnswered && answerType,
            'incorrect-answer': isAnswered && !answerType
          }"
          v-for="(value, index) in handValues"
          :key="index"
          @click="handleAnswer(value)"
          :disabled="!timerSeconds || isAnswered"
        >
          {{ value }}
        </button>
      </div>
      <PokerTable :playerCards="playerCards" :communityCards="communityCards" />
    </section>
  </main>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: row;
  width: 100%;
  padding: 10px;
}
.info-data {
  display: flex;
  justify-content: center;
  flex-direction: column;
  min-width: 200px;
}
.points {
  font-size: 40px;
  font-weight: bold;
}

.timer {
  font-size: 40px;
  font-weight: bold;
}
.hand-values-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 10px 0;
}
.hand-values {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
  margin: 10px 0;
}

.modal-text {
  font-size: 30px;
  font-weight: bold;
  margin: 10px 0;
  color: var(--dark-color);
}

.btn {
  width: 150px;
  height: 35px;
  background-color: var(--light-color);
  border: none;
  border-radius: 5px;
  padding: 10px;
  margin: 0 5px;
  cursor: pointer;
  box-shadow: 0 0 10px var(--dark-color);
}

.correct-answer {
  outline: 4px solid var(--success-color);
}

.incorrect-answer {
  outline: 4px solid var(--danger-color);
}
</style>
