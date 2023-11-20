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

enum HandValues {
  HighCard = 'High Card',
  Pair = 'Pair',
  TwoPair = 'Two Pair',
  ThreeOfAKind = 'Three of a Kind',
  Straight = 'Straight',
  Flush = 'Flush',
  FullHouse = 'Full House',
  FourOfAKind = 'Four of a Kind',
  StraightFlush = 'Straight Flush',
  RoyalFlush = 'Royal Flush'
}
const handValue = ref<string>('')
const points = ref(10)
const isAnswered = ref(false)
const timerSeconds = ref<number>(10)
const answerType = ref<null | boolean>(null)
const showModal = ref(true)
const showModal2 = ref(false)
let startTime = ref<any | number>(null)
let totalTime = ref<any | number>(null)
let intervalId: any = null
let roundNumber = ref<number>(0)
const apiData = ref<any>(null)

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
  const deckCopy = [...deck.value]
  playerCards.value = deckCopy.splice(0, 2)
  deckCopy.splice(0, 1)
  communityCards.value = deckCopy.splice(0, 3)
  deckCopy.splice(0, 1)
  communityCards.value.push(...deckCopy.splice(0, 1))
  deckCopy.splice(0, 1)
  communityCards.value.push(...deckCopy.splice(0, 1))
}

const getBestHand = () => {
  const allCards = [...playerCards.value, ...communityCards.value]

  const hands = allCards.map((card) => `${card.rank}${card.suit[0].toLowerCase()}`)
  handValue.value = PokerSolver.Hand.solve(hands).name
}

const handleAnswer = (value: String) => {
  isAnswered.value = true
  clearInterval(intervalId)

  if (handValue.value && value === handValue.value) {
    answerType.value = true
    points.value += 1
  } else {
    answerType.value = false
    points.value -= points.value >= 3 ? 3 : points.value
  }
}

const startGame = () => {
  showModal.value = false
  startTime.value = Date.now()
  points.value = 10
  timerSeconds.value = 10
  createDeck()
  shuffleDeck()
  dealCards()
  getBestHand()

  if (handValue.value) {
    startRound()
  }
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

watch(timerSeconds, (newValue) => {
  if (timerSeconds.value <= 0) {
    totalTime.value = Date.now() - startTime.value
    let timeInSeconds = totalTime.value / 1000
    let minutes = Math.floor(timeInSeconds / 60)
    let seconds = Math.floor(timeInSeconds % 60)
    alert(`Time spent: ${minutes} minutes and ${seconds} seconds`)
    clearInterval(intervalId)
  }
})

async function fetchData() {
  const randomNumber = Math.floor(Math.random() * 200) + 1
  const url = `https://jsonplaceholder.typicode.com/todos/${randomNumber}`
  try {
    const response = await fetch(url)

    apiData.value = await response.json()
    showModal2.value = true
  } catch (error) {
    console.error(error)
  }
}

const startRound = () => {
  isAnswered.value = false
  shuffleDeck()
  dealCards()
  getBestHand()
  let timer = timerSeconds.value

  if (handValue.value) {
    intervalId = setInterval(() => {
      if (timer > 0) {
        timer--
      } else {
        points.value -= points.value >= 3 ? 3 : points.value
        clearInterval(intervalId)
      }
    }, 1000)
  }
}

const restartRound = () => {
  showModal2.value = false
  if (points.value > 0) {
    clearInterval(intervalId)
    roundNumber.value = roundNumber.value + 1
    timerSeconds.value = 10 - roundNumber.value
    isAnswered.value = false
    startRound()
  }
}
</script>

<template>
  <StartModal v-model="showModal2" v-if="showModal2">
    <h2 class="">{{ apiData.title }}</h2>
    <button class="btn" @click="restartRound">Continue</button>
  </StartModal>
  <StartModal v-model="showModal" v-if="showModal">
    <h2>Start a new game?</h2>
    <button class="btn" @click="startGame">Start</button>
  </StartModal>
  <main
    class="container"
    :class="!timerSeconds && 'incorrect-answer'"
    v-if="!showModal2 || showModal"
  >
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
          v-for="(value, index) in HandValues"
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
