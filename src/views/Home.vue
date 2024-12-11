<script setup>
  import { ref } from 'vue'

  const deck = ref([
    '2@',
    '2#',
    '2^',
    '2*',
    '3@',
    '3#',
    '3^',
    '3*',
    '4@',
    '4#',
    '4^',
    '4*',
    '5@',
    '5#',
    '5^',
    '5*',
    '6@',
    '6#',
    '6^',
    '6*',
    '7@',
    '7#',
    '7^',
    '7*',
    '8@',
    '8#',
    '8^',
    '8*',
    '9@',
    '9#',
    '9^',
    '9*',
    '10@',
    '10#',
    '10^',
    '10*',
    'J@',
    'J#',
    'J^',
    'J*',
    'Q@',
    'Q#',
    'Q^',
    'Q*',
    'K@',
    'K#',
    'K^',
    'K*',
    'A@',
    'A#',
    'A^',
    'A*',
  ])

  const players = ref([[], [], [], []])
  const winner = ref(null)
  const ifStarted = ref(false)
  const gameState = ref('idle')

  // 洗牌
  const shuffleDeck = deck => {
    for (let i = deck.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1))
      ;[deck[i], deck[j]] = [deck[j], deck[i]]
    }
    return deck
  }
  const shuffleCards = () => {
    deck.value = shuffleDeck([...deck.value])
    gameState.value = 'idle'
    ifStarted.value = false
    winner.value = null
  }

  // 分发卡牌
  const dealCards = deck => {
    const players = [[], [], [], []]
    deck.forEach((card, index) => {
      players[index % 4].push(card)
    })
    ifStarted.value = true
    winner.value = null
    return players
  }
  const deal = () => {
    players.value = dealCards(deck.value)
    gameState.value = 'playing'
  }

  // 确定赢家
  const compareCardValues = (cardA, cardB) => {
    const valuesOrder = [
      '2',
      '3',
      '4',
      '5',
      '6',
      '7',
      '8',
      '9',
      '10',
      'J',
      'Q',
      'K',
      'A',
    ]
    return valuesOrder.indexOf(cardA) - valuesOrder.indexOf(cardB)
  }
  const determineWinnerHandler = players => {
    const playerGroups = players.map(player => {
      const groups = {}
      player.forEach(card => {
        const cardValue = card.slice(0, -1)
        if (!groups[cardValue]) {
          groups[cardValue] = []
        }
        groups[cardValue].push(card)
      })
      return groups
    })

    let maxGroupSize = 0
    let winnerIndex = -1

    playerGroups.forEach((groups, playerIndex) => {
      Object.values(groups).forEach(group => {
        if (group.length > maxGroupSize) {
          maxGroupSize = group.length
          winnerIndex = playerIndex
        } else if (group.length === maxGroupSize) {
          // 如果组的大小相同，则比较字母数字的大小
          const currentWinnerCardValue = playerGroups[winnerIndex][0]?.slice(
            0,
            -1,
          )
          const newPlayerCardValue = group[0]?.slice(0, -1)
          if (compareCardValues(newPlayerCardValue, currentWinnerCardValue) > 0) {
            winnerIndex = playerIndex
          }
        }
      })
    })

    return winnerIndex
  }

  const determineWinner = () => {
    if (!ifStarted.value) {
      winner.value = '请先发牌！'
      return
    }
    const winnerIndex = determineWinnerHandler(players.value)
    winner.value = `Player ${winnerIndex + 1} is the winner!`
    gameState.value = 'finished'
  }
  </script>

  <template>
    <div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
      <h1 class="text-2xl font-bold mb-4">Card Game</h1>
        <div>
          <button
            class="bg-blue-500 text-white py-2 px-4 rounded mr-2"
            @click="shuffleCards"
          >
            Shuffle Deck
          </button>
          <button
            class="bg-blue-500 text-white py-2 px-4 rounded mr-2"
            @click="deal"
          >
            Deal Cards
          </button>
          <button
            class="bg-blue-500 text-white py-2 px-4 rounded"
            @click="determineWinner"
            :disabled="!ifStarted"
            :class="{ 'opacity-50 cursor-not-allowed': !ifStarted }"
          >
            Determine Winner
          </button>
        </div>
        <div class="players-container">
          <div v-for="(player, index) in players" :key="index" class="player-section">
            <h3 v-if="ifStarted" class="player-title">Player {{ index + 1 }}'s Cards</h3>
            <ul v-if="ifStarted" class="cards-list">
              <li v-for="card in player" :key="card" class="card-item">
                <div class="card transition-transform duration-500 transform hover:scale-105">
                  {{ card }}
                </div>
              </li>
            </ul>
          </div>
        </div>
      <h2 v-if="winner && ifStarted" class="text-red-500 text-xl mt-10">{{ winner }}</h2>
      <h2 v-if="winner && !ifStarted" class="text-xl mt-10">Cards have been shuffled.</h2>
    </div>
  </template>

  <style scoped>
  @import 'tailwindcss/tailwind.css';

  .players-container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 2rem;
  }

  .player-section {
    flex: 1;
    min-width: 200px;
    text-align: center;
  }

  .player-title {
    font-size: 1.25rem;
    margin: 0.5rem 0;
    text-align: center;
  }

  .cards-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .card-item {
    margin: 0.5rem;
    padding: 0.5rem;
    background-color: #f0f0f0;
    border-radius: 0.25rem;
  }

  .card {
    position: relative;
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .card[data-suit="@"] { color: #e53e3e; }
  .card[data-suit="#"] { color: #e53e3e; }
  .card[data-suit="^"] { color: #2d3748; }
  .card[data-suit="*"] { color: #2d3748; }
  </style>
