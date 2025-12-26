<template>
    <div class="az-card" @click="onClick">
        <div class="az-card__inner" :style="flipStyle">
            <div class="az-card__front"></div>
            <div class="az-card__back" :style="backStyle"></div>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { computed } from 'vue'

const props = defineProps({
    revealed: {
        type: Boolean,
        required: true
    },
    image: {
        type: String,
        required: true
    }
})

const emit = defineEmits(['click'])

const onClick = () => {
    emit('click')
}

const flipStyle = computed(() => ({
    transform: props.revealed ? 'rotateY(180deg)' : 'rotateY(0deg)'
}))

const backStyle = computed(() => ({
    backgroundImage: `url(${new URL(`/src/assets/${props.image}`, import.meta.url).href})`
}))
</script>

<style lang="scss" scoped>
.az-card {
    width: 120px;
    height: 120px;
    perspective: 600px;
    cursor: pointer;

    &__inner {
        position: relative;
        width: 100%;
        height: 100%;
        transform-style: preserve-3d;
        transition: transform 0.35s ease;
    }

    &__front,
    &__back {
        position: absolute;
        width: 100%;
        height: 100%;
        border-radius: 12px;
        backface-visibility: hidden;
    }

    &__front {
        background-image: url('@/assets/mermid-card.png');
        background-size: cover;
        background-position: center;
    }

    &__back {
        transform: rotateY(180deg);
        background-size: cover;
        background-position: center;
    }
}
</style>
