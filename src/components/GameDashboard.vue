<template>
    <div class="az-dashboard">
        <div class="az-dashboard__time">
            Time: {{ time }}s
        </div>

        <div v-if="paused" class="az-dashboard__paused">
            Paused
        </div>

        <div
            v-else-if="mode === 'vsAI' && showTurn"
            class="az-dashboard__turn"
            :class="'az-dashboard__turn--'+currentTurn"
        >
            <div class="az-dashboard__turn-indicator"></div>
            {{ currentTurn === 'player' ? 'Your turn' : 'AI’s turn' }}
        </div>

        <div class="az-dashboard__moves">
            Moves: {{ moves }}
        </div>
    </div>
</template>

<script lang="ts" setup>
defineProps<{
    time: number
    moves: number
    paused: boolean
    mode: 'solo' | 'vsAI'
    currentTurn: 'player' | 'ai'
    showTurn: boolean
}>()
</script>

<style lang="scss" scoped>
.az-dashboard {
    width: 100%;
    height: 24px;
    padding: 0 8px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: $color-grey-200;
    font-size: 13px;
    font-weight: 400;
    color: $color-grey-400;

    &__time {
        width: 120px;
        text-align: left;
    }

    &__paused {
        font-weight: 700;
        text-align: center;
        text-transform: uppercase;
    }

    &__turn {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        gap: 8px;

        &-indicator {
            width: 10px;
            height: 10px;
            border-radius: 50%;
        }

        &--player {
            .az-dashboard__turn-indicator {
                background: $color-rose-200;
            }
        }

        &--ai {
            .az-dashboard__turn-indicator {
                background: $color-mint-200;
            }
        }
    }

    &__moves {
        width: 120px;
        text-align: right;
    }
}
</style>