<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import PokerTable from './components/PokerTable.vue'
import StartModal from './components/StartModal.vue'
import * as PokerSolver from 'pokersolver'

export type Card = {
  rank: string
  suit: string
}

const playerCards = ref<Card[]>([])

const communityCards = ref<Card[]>([])

const suits = ['H', 'D', 'C', 'S']
const ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A']
const deck = ref<Card[]>([])

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
const showModal = ref(false)
const showModal2 = ref(false)
let startTime = ref<any | number>(null)
let totalTime = ref<any | number>(null)
let intervalId = ref<any>(null)
let roundNumber = ref<number>(0)
const apiData = ref<any>(null)
const scoreData = ref<any>(null)
const answerType = ref<string>('')
const gameHistory = ref<any[]>([])

const createDeck = () => {
  deck.value = []
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
const reset = () => {
  startTime.value = Date.now()
  showModal.value = false
  scoreData.value = null
  isAnswered.value = false
  answerType.value = ''
  apiData.value = null
  scoreData.value = null
  points.value = 10
  timerSeconds.value = 10
  roundNumber.value = 0
  clearInterval(intervalId.value)
}
let isGameRestarting = false
const startGame = () => {
  isGameRestarting = true
  reset()
  createDeck()
  gameHistory.value = []
  if (deck.value.length) {
    startRound()
  }
  setTimeout(() => {
    isGameRestarting = false
  }, 1000)
}

const startRound = () => {
  isAnswered.value = false
  answerType.value = ''
  apiData.value = null
  handValue.value = ''
  timerSeconds.value = 10
  shuffleDeck()
  dealCards()
  getBestHand()
  gameHistory.value.push({
    playerCards: [...playerCards.value],
    communityCards: [...communityCards.value],
    bestHand: handValue.value
  })

  timerSeconds.value = Math.max(timerSeconds.value - roundNumber.value, 3)
  intervalId.value = setInterval(() => {
    if (timerSeconds.value > 0) {
      timerSeconds.value--
    } else {
      points.value -= points.value >= 3 ? 3 : points.value
    }
  }, 1200)
}

const handleAnswer = (answer: String) => {
  if (answer === handValue.value) {
    answerType.value = 'correct-answer'
    points.value += 1
  } else {
    answerType.value = 'incorrect-answer'

    points.value -= points.value >= 3 ? 3 : points.value
  }
}

const checkPoints = () => {
  if (isGameRestarting) {
    return
  }
  if (points.value <= 0) {
    totalTime.value = Date.now() - startTime.value
    let timeInSeconds = totalTime.value / 1000
    let minutes = Math.floor(timeInSeconds / 60)
    let seconds = Math.floor(timeInSeconds % 60)
    apiData.value = null
    scoreData.value = {
      time: `Time spent: ${minutes} minutes and ${seconds} seconds `,
      gameHistory: [...gameHistory.value]
    }
    showModal2.value = true
    clearInterval(intervalId.value)
  } else {
    roundNumber.value += 1
    clearInterval(intervalId.value)
    fetchData().then(() => {
      showModal2.value = true
    })
  }
}

const responseClass = computed(() => {
  switch (answerType.value) {
    case 'correct-answer':
      return 'correct-answer'
    case 'incorrect-answer':
      return 'incorrect-answer'
    default:
      return ''
  }
})

async function fetchData() {
  const randomNumber = Math.floor(Math.random() * 200) + 1
  const url = `https://jsonplaceholder.typicode.com/todos/${randomNumber}`
  try {
    const response = await fetch(url)

    apiData.value = await response.json()
  } catch (error) {
    console.error(error)
  }
}

const handleInfoModal = () => {
  if (!points.value) {
    clearInterval(intervalId.value)
    startGame()
  } else {
    clearInterval(intervalId.value)

    startRound()
  }
  showModal2.value = false
}

onMounted(() => {
  showModal.value = true
})

watch(points, checkPoints)
</script>

<template>
  <StartModal v-model="showModal2" v-if="showModal2">
    <h2 v-if="apiData">{{ apiData?.title }}</h2>
    <div v-if="scoreData">
      <h2 class="modal-text">Game History</h2>
      <p class="modal-text">{{ scoreData.time }}</p>
      <table class="game-history-table">
        <thead>
          <tr>
            <th>Round</th>
            <th>Player Cards</th>
            <th>Community Cards</th>
            <th>Best Hand</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(round, index) in scoreData.gameHistory" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ round.playerCards.map((card: Card) => card.rank + card.suit).join(', ') }}</td>
            <td>
              {{ round.communityCards.map((card: Card) => card.rank + card.suit).join(', ') }}
            </td>
            <td>{{ round.bestHand }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <button class="btn" @click="handleInfoModal">Play game</button>
  </StartModal>
  <StartModal v-model="showModal" v-if="showModal">
    <h2>Guess the hand!</h2>
    <button class="btn" @click="startGame">Start game</button>
  </StartModal>
  <main class="container" :class="!timerSeconds && 'incorrect-answer'">
    <section class="info-data">
      <p class="points">Stamina: {{ points }}</p>
      <p class="timer">Timer: {{ timerSeconds }}</p>
    </section>
    <section class="hand-values-container">
      <h2>Guess your hand!</h2>
      <div class="hand-values">
        <button
          class="btn"
          :class="responseClass"
          v-for="(value, index) in HandValues"
          :key="index"
          @click="handleAnswer(value)"
          :disabled="isAnswered || !timerSeconds"
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
  justify-content: center;
  align-items: center;
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
.game-history-table {
  width: 100%;
  border-collapse: collapse;
}

.game-history-table th,
.game-history-table td {
  border: 1px solid #ddd;
  padding: 8px;
}

.game-history-table tr:nth-child(even) {
  background-color: #f2f2f2;
}

.game-history-table th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #4caf50;
  color: white;
}
</style>
