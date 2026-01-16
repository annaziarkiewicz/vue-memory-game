<template>
    <div class="az-game-screen">
        <div class="az-game-screen__box">
            <template v-if="screen === 'intro'">
                <div class="az-game-screen__heading">
                    Vue.js Memory game
                </div>

                <div class="az-game-screen__subtitle">
                    This is a classic card-matching game played on a 6×4 grid.<br/>
                    The goal is to find and match pairs of identical cards.
                </div>
            </template>

            <template v-else>
                <div class="az-game-screen__heading">
                    {{ winner }}
                </div>

                <div class="az-game-screen__subtitle">
                    Time: <b>{{ time }}s</b> |  Moves: <b>{{ moves }}</b>
                </div>
            </template>

            <div class="az-game-screen__modes">
                <button
                    class="az-game-screen__button az-game-screen__button--rose"
                    @click="$emit('play', 'solo')">
                    Play Solo
                </button>

                <button
                    class="az-game-screen__button az-game-screen__button--mint"
                    @click="$emit('play', 'vsAI')">
                    Play vs AI
                </button>
            </div>
        </div>
    </div>
</template>

<script lang="ts" setup>
defineProps<{
    screen: 'intro' | 'summary'
    winner?: string
    time?: number
    moves?: number
}>()

defineEmits(['play'])
</script>

<style lang="scss" scoped>
.az-game-screen {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 1;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 32px;
    background: rgba(175,175,168, .5);

    &__box {
        position: relative;
        width: 480px;
        height: 320px;
        padding: 0 64px;
        margin-top: 24px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 12px;
        background: $color-milk-200;
        box-shadow: 0 0 24px $color-grey-400;
        border-radius: 24px;

        &:before,
        &:after {
            content: '';
            position: absolute;
            display: block;
            pointer-events: none;
        }

        &:before {
            top: -116px;
            left: -111px;
            width: 236px;
            height: 318px;
            background-image: url('@/assets/mermid-before.png');
        }

        &:after {
            bottom: -116px;
            right: -147px;
            width: 292px;
            height: 294px;
            background-image: url('@/assets/mermid-after.png');
        }
    }

    &__heading {
        font-size: 32px;
        font-weight: 700;
        line-height: 1em;
        color: $color-milk-500;
    }

    &__subtitle {
        font-size: 13px;
        font-weight: 200;
        line-height: 1.25rem;
        color: $color-grey-400;
        text-align: center;
    }

    &__modes {
        width: 100%;
        display: flex;
        flex-direction: column;
        gap: 12px;
    }

    &__button {
        width: 100%;
        padding: 12px;
        border-radius: 12px;
        font-size: 18px;
        font-weight: 500;
        text-transform: uppercase;
        transition: 0.5s;

        &--rose {
            background: $color-rose-200;

            &:hover {
                background: $color-rose-300;
            }
        }

        &--mint {
            background: $color-mint-400;

            &:hover {
                background: $color-mint-500;
            }
        }
    }
}
</style>
