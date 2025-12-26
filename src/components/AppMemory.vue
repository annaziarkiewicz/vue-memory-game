<template>
    <section class="az-game">
        <GameDashboard
            :time="time"
            :moves="moves"
            :paused="paused"
            :mode="mode"
            :currentTurn="currentTurn"
            :showTurn="!showWelcome"
        />

        <div class="az-game__board">
            <GameCard
                v-for="(card, index) in cards"
                :key="index"
                :revealed="card.revealed"
                :image="card.image"
                @click="onCardClick(index)"
            />
        </div>

        <GameWelcomeScreen
            v-if="showWelcome"
            :screen="screen"
            :winner="winnerMessage"
            :time="time"
            :moves="moves"
            @play="startGame"
        />
    </section>
</template>

<script lang="ts" setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import GameCard from '@/components/GameCard.vue'
import GameDashboard from '@/components/GameDashboard.vue'
import GameWelcomeScreen from '@/components/GameWelcomeScreen.vue'

type Mode = 'solo' | 'vsAI'
type Turn = 'player' | 'ai'

const showWelcome = ref(true)
const screen = ref<'intro' | 'summary'>('intro')

const mode = ref<Mode>('solo')
const currentTurn = ref<Turn>('player')

const cards = ref<{ image: string; revealed: boolean }[]>([])
const flipped = ref<number[]>([])
const matched = ref<number[]>([])

const moves = ref(0)
const time = ref(0)
const paused = ref(true)
const resolving = ref(false)

const playerPairs = ref(0)
const aiPairs = ref(0)

let timerInterval: number | null = null

type MemoryCard = { index: number; image: string }
const aiMemory = ref<MemoryCard[]>([])

const rememberCard = (index: number, image: string) => {
    if (aiMemory.value.find(c => c.index === index)) return
    aiMemory.value.push({ index, image })
    if (aiMemory.value.length > 6) aiMemory.value.shift()
}

const forgetMatched = (i1: number, i2: number) => {
    aiMemory.value = aiMemory.value.filter(
        c => c.index !== i1 && c.index !== i2
    )
}

const findKnownPair = (): [number, number] | null => {
    for (let i = 0; i < aiMemory.value.length; i++) {
        for (let j = i + 1; j < aiMemory.value.length; j++) {
            if (aiMemory.value[i].image === aiMemory.value[j].image) {
                return [aiMemory.value[i].index, aiMemory.value[j].index]
            }
        }
    }
    return null
}

const winnerMessage = computed(() => {
    if (mode.value === 'solo') return 'Your score'
    if (playerPairs.value > aiPairs.value) return 'You win'
    if (playerPairs.value < aiPairs.value) return 'AI wins'
    return 'Draw'
})

const startGame = (m: Mode) => {
    mode.value = m
    showWelcome.value = false
    screen.value = 'intro'

    paused.value = false
    resolving.value = false
    currentTurn.value = 'player'

    moves.value = 0
    time.value = 0
    playerPairs.value = 0
    aiPairs.value = 0

    flipped.value = []
    matched.value = []
    aiMemory.value = []

    prepareCards()
    startTimer()
}

const endGame = () => {
    paused.value = true
    resolving.value = false

    cards.value.forEach(card => (card.revealed = true))

    screen.value = 'summary'
    showWelcome.value = true

    if (timerInterval) clearInterval(timerInterval)
}

const startTimer = () => {
    if (timerInterval) clearInterval(timerInterval)
    timerInterval = window.setInterval(() => {
        if (!paused.value) time.value++
    }, 1000)
}

const prepareCards = () => {
    const images: string[] = []

    for (let i = 1; i <= 12; i++) {
        const name = `mermid-${String(i).padStart(2, '0')}.png`
        images.push(name, name)
    }

    cards.value = images
        .sort(() => Math.random() - 0.5)
        .map(image => ({ image, revealed: false }))
}

const onCardClick = (index: number) => {
    if (paused.value) return
    if (resolving.value) return
    if (mode.value === 'vsAI' && currentTurn.value !== 'player') return
    if (flipped.value.length === 2) return

    const card = cards.value[index]
    if (!card || card.revealed) return

    card.revealed = true
    flipped.value.push(index)
    rememberCard(index, card.image)

    if (flipped.value.length === 2) {
        moves.value++
        resolveTurn('player')
    }
}

const resolveTurn = (who: Turn) => {
    resolving.value = true

    const [i1, i2] = flipped.value
    const c1 = cards.value[i1]
    const c2 = cards.value[i2]

    if (c1.image === c2.image) {
        matched.value.push(i1, i2)
        forgetMatched(i1, i2)

        if (who === 'player') playerPairs.value++
        else aiPairs.value++

        flipped.value = []
        resolving.value = false

        if (matched.value.length === cards.value.length) {
            currentTurn.value = 'player'
            endGame()
            return
        }

        if (who === 'ai') {
            currentTurn.value = 'ai'
            setTimeout(aiMove, 500)
        }

        return
    }

    setTimeout(() => {
        c1.revealed = false
        c2.revealed = false
        flipped.value = []
        resolving.value = false

        if (mode.value === 'vsAI') {
            currentTurn.value = who === 'player' ? 'ai' : 'player'
            if (currentTurn.value === 'ai') {
                setTimeout(aiMove, 500)
            }
        }
    }, 900)
}

const aiMove = () => {
    if (paused.value) return
    if (resolving.value) return
    if (currentTurn.value !== 'ai') return

    const known = findKnownPair()
    if (known) {
        aiFlip(known[0], known[1])
        return
    }

    const hidden = cards.value
        .map((_, i) => i)
        .filter(i => !cards.value[i].revealed && !matched.value.includes(i))

    const unknown = hidden.filter(
        i => !aiMemory.value.find(m => m.index === i)
    )

    const pool = unknown.length ? unknown : hidden
    if (pool.length < 2) return

    const i1 = pool[Math.floor(Math.random() * pool.length)]
    let i2 = pool[Math.floor(Math.random() * pool.length)]
    while (i2 === i1) {
        i2 = pool[Math.floor(Math.random() * pool.length)]
    }

    aiFlip(i1, i2)
}

const aiFlip = (i1: number, i2: number) => {
    cards.value[i1].revealed = true
    rememberCard(i1, cards.value[i1].image)

    setTimeout(() => {
        cards.value[i2].revealed = true
        rememberCard(i2, cards.value[i2].image)

        flipped.value = [i1, i2]
        moves.value++

        setTimeout(() => resolveTurn('ai'), 400)
    }, 400)
}

const onKeydown = (e: KeyboardEvent) => {
    if (e.code !== 'Space') return
    if (showWelcome.value) return
    if (resolving.value) return
    paused.value = !paused.value
}

onMounted(() => {
    prepareCards()
    window.addEventListener('keydown', onKeydown)
})

onUnmounted(() => {
    window.removeEventListener('keydown', onKeydown)
})
</script>

<style lang="scss" scoped>
.az-game {
    position: relative;
    margin: 32px 0;
    display: flex;
    flex-direction: column;
    align-items: center;

    &__board {
        padding: 24px;
        display: grid;
        grid-template-columns: repeat(6, 120px);
        grid-template-rows: repeat(4, 120px);
        gap: 24px;
        background: $color-milk-200;
    }
}
</style>
