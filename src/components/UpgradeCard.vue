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
import { computed } from 'vue'

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
.upgrade-card {
  background-color: #2d2d2d;
  border-radius: 16px;
  padding: 16px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;

  &__info {
    margin-bottom: 16px;
  }

  &__title {
    font-size: 1.25rem;
    font-weight: bold;
    color: #ffffff;
  }

  &__description {
    font-size: 0.875rem;
    color: #b3b3b3;
  }

  &__action {
    margin-top: 16px;
  }

  &__button {
    width: 100%;
    background-color: #4caf50;
    color: #ffffff;
    padding: 12px;
    border-radius: 8px;
    font-size: 1rem;
    font-weight: bold;
    transition: background-color 0.3s ease;
    cursor: pointer;

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
