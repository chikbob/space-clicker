<template>
  <div class="upgrade-card">
    <div class="upgrade-card__info">
      <h3 class="upgrade-card__title">{{ upgrade.name }}</h3>
      <p class="upgrade-card__description">{{ upgrade.description }}</p>
    </div>
    <div class="upgrade-card__action">
      <button
          @click="props.buyUpgrade(upgrade)"
          :disabled="!canAfford"
          class="upgrade-card__button"
      >
        Купить ({{ upgrade.level }} ур.) - {{ upgrade.cost }} очков
      </button>
    </div>
  </div>
</template>

<script setup>
import {computed} from 'vue'

// Получаем пропсы
const props = defineProps({
  upgrade: {
    type: Object,
    required: true
  },
  buyUpgrade: Function,
  score: {
    type: Number,
    required: true
  }
})

// Вычисляем возможность покупки
const canAfford = computed(() => {
  return props.score >= props.upgrade.cost
})
</script>

<style scoped lang="scss">
$breakpoints: (
    sm: 480px,
    md: 768px,
    lg: 1024px
);

@mixin respond-above($breakpoint) {
  @media (min-width: map-get($breakpoints, $breakpoint)) {
    @content;
  }
}

.upgrade-card {
  background-color: #2d2d2d;
  border-radius: 12px;
  padding: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-sizing: border-box;

  &__info {
    margin-bottom: 10px;
  }

  &__title {
    font-size: clamp(1rem, 3vw, 1.3rem);
    font-weight: bold;
    color: #ffffff;
    word-break: break-word;
  }

  &__description {
    font-size: clamp(0.8rem, 2.5vw, 0.95rem);
    color: #b3b3b3;
  }

  &__action {
    margin-top: 10px;
  }

  &__button {
    width: 100%;
    background-color: #4caf50;
    color: #ffffff;
    padding: 10px;
    border-radius: 6px;
    font-size: clamp(0.85rem, 3vw, 1rem);
    font-weight: bold;
    transition: background-color 0.3s ease;
    cursor: pointer;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;

    &:hover {
      background-color: #45a049;
    }

    &:disabled {
      background-color: #757575;
      cursor: not-allowed;
    }
  }
}
</style>