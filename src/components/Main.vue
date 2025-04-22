<template>
  <div class="page__container">
    <!-- Верхня панель з очками та кнопкою -->
    <div class="page__header">
      <h1 class="page__title">🚀 Космічний клікер</h1>
      <p class="page__score">Очки: {{ score }}</p>
      <button @click="clickHandler" class="page__button">
        Клік!
      </button>
    </div>

    <!-- Нижня секція: покращення -->
    <div class="page__content">
      <!-- Ліва колонка: Активні покращення -->
      <div class="page__section page__section--active">
        <h2 class="page__section-title page__section-title--active">
          🔨 Активні покращення
        </h2>
        <div class="page__upgrades">
          <UpgradeCard
              v-for="upgrade in activeUpgrades"
              :key="upgrade.id"
              :upgrade="upgrade"
              :score="score"
              :buy-upgrade="buyClickUpgrade"
          />
        </div>
      </div>

      <!-- Права колонка: Пасивні покращення -->
      <div class="page__section page__section--passive">
        <h2 class="page__section-title page__section-title--passive">
          ⚙️ Пасивні покращення
        </h2>
        <div class="page__upgrades">
          <UpgradeCard
              v-for="upgrade in visiblePassiveUpgrades"
              :key="upgrade.id"
              :upgrade="upgrade"
              :score="score"
              :buy-upgrade="buyPassiveUpgrade"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref, computed, onMounted, watch} from 'vue';
import {defineStore} from 'pinia'
import UpgradeCard from './UpgradeCard.vue';

const score = ref(0);

const clickUpgrades = ref([
  {id: 1, name: '👨‍🚀 Космонавт', level: 0, value: 1, baseCost: 50, cost: 50, locked: false},
  {id: 2, name: '🚀 Шаттл', level: 0, value: 5, baseCost: 300, cost: 300, locked: true},
  {id: 3, name: '🛰️ Супутник', level: 0, value: 15, baseCost: 1000, cost: 1000, locked: true},
  {id: 4, name: '🌌 Зоряний крейсер', level: 0, value: 40, baseCost: 5000, cost: 5000, locked: true},
  {id: 5, name: '👾 Інопланетний союзник', level: 0, value: 100, baseCost: 20000, cost: 20000, locked: true},
]);

const passiveUpgrades = ref([
  {id: 100, name: '🔌 Сонячна батарея', level: 0, value: 1, baseCost: 100, cost: 100, locked: false},
  {id: 101, name: '🏭 Космостанція', level: 0, value: 5, baseCost: 500, cost: 500, locked: true},
  {id: 102, name: '🧠 ШІ-Аналітик', level: 0, value: 15, baseCost: 2000, cost: 2000, locked: true},
  {id: 103, name: '⚡ Темна енергія', level: 0, value: 50, baseCost: 8000, cost: 8000, locked: true},
  {id: 104, name: '💠 Квантовий генератор', level: 0, value: 150, baseCost: 25000, cost: 25000, locked: true},
]);

const totalClickValue = computed(() => {
  return clickUpgrades.value.reduce((sum, u) => sum + u.value * u.level, 0);
});

const totalPassiveIncome = computed(() => {
  return passiveUpgrades.value.reduce((sum, u) => sum + u.value * u.level, 0);
});

function unlockNextClickUpgrade() {
  clickUpgrades.value.forEach((upgrade, index) => {
    if (index > 0 && clickUpgrades.value[index - 1].level >= 5) {
      upgrade.locked = false;
    }
  });
}

function unlockNextPassiveUpgrade() {
  passiveUpgrades.value.forEach((upgrade, index) => {
    if (index > 0 && passiveUpgrades.value[index - 1].level >= 5) {
      upgrade.locked = false;
    }
  });
}

function buyClickUpgrade(upgrade) {
  if (score.value >= upgrade.cost) {
    score.value -= upgrade.cost;
    upgrade.level++;
    upgrade.cost = Math.floor(upgrade.baseCost * Math.pow(1.5, upgrade.level));
    unlockNextClickUpgrade();
  }
}

function buyPassiveUpgrade(upgrade) {
  if (score.value >= upgrade.cost) {
    score.value -= upgrade.cost;
    upgrade.level++;
    upgrade.cost = Math.floor(upgrade.baseCost * Math.pow(1.5, upgrade.level));
    unlockNextPassiveUpgrade(); // <== додано
  }
}

function clickHandler() {
  const baseClick = 1;
  score.value += baseClick + totalClickValue.value;
}

function loadFromStorage() {
  const data = localStorage.getItem('spaceClicker');
  if (data) {
    const parsed = JSON.parse(data);
    score.value = parsed.score;

    clickUpgrades.value = parsed.clickUpgrades.map((u, i) => ({
      ...clickUpgrades.value[i],
      ...u,
    }));

    passiveUpgrades.value = parsed.passiveUpgrades.map((u, i) => ({
      ...passiveUpgrades.value[i],
      ...u,
    }));
  }
}

function saveToStorage() {
  localStorage.setItem('spaceClicker', JSON.stringify({
    score: score.value,
    clickUpgrades: clickUpgrades.value,
    passiveUpgrades: passiveUpgrades.value,
  }));
}

const activeUpgrades = computed(() => {
  return clickUpgrades.value.filter(u => !u.locked);
});

onMounted(() => {
  loadFromStorage();
  setInterval(() => {
    score.value += totalPassiveIncome.value;
  }, 1000);
});

watch([score, clickUpgrades, passiveUpgrades], saveToStorage, {deep: true});

const visiblePassiveUpgrades = computed(() => {
  return passiveUpgrades.value.filter(u => !u.locked);
});

</script>

<style scoped lang="scss">
$page-breakpoints: (
    sm: 480px,
    md: 768px,
    lg: 1024px,
    xl: 1280px
);

@mixin respond-above($breakpoint) {
  @media (min-width: map-get($page-breakpoints, $breakpoint)) {
    @content;
  }
}

.page {
  &__container {
    background-color: #1e1e2f;
    color: #ffffff;
    padding: 16px;
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    box-sizing: border-box;
  }

  &__header {
    text-align: center;
    margin-bottom: 20px;
  }

  &__title {
    font-size: clamp(1.6rem, 5vw, 3rem);
    font-weight: bold;
    margin-bottom: 16px;
    word-wrap: break-word;
  }

  &__score {
    font-size: clamp(1.1rem, 4vw, 2rem);
    font-weight: 600;
    margin-bottom: 16px;
  }

  &__button {
    background-color: #9b4dca;
    color: #ffffff;
    padding: 14px 28px;
    border-radius: 16px;
    font-size: clamp(1rem, 3vw, 1.4rem);
    font-weight: bold;
    transition: background-color 0.3s ease;
    margin: 0 auto 24px;
    max-width: 90%;
    width: 100%;

    @include respond-above(md) {
      width: 280px;
    }

    &:hover {
      background-color: #6a2c9f;
    }
  }

  &__content {
    display: flex;
    flex-direction: column;
    gap: 1.2rem;
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 0;
    box-sizing: border-box;

    @include respond-above(md) {
      flex-direction: row;
      flex-wrap: wrap;
      justify-content: center;
    }
  }

  &__section {
    background-color: #33333d;
    border-radius: 16px;
    padding: 16px;
    width: 100%;
    box-sizing: border-box;

    @include respond-above(sm) {
      flex: 1 1 320px;
      max-width: 500px;
    }
  }

  &__section-title {
    font-size: clamp(1.3rem, 4vw, 2rem);
    font-weight: 600;
    border-bottom: 2px solid;
    padding-bottom: 8px;
    margin-bottom: 16px;
    text-align: center;

    &--active {
      color: #9b4dca;
      border-color: #9b4dca;
    }

    &--passive {
      color: #3b8bbd;
      border-color: #3b8bbd;
    }
  }

  &__upgrades {
    display: flex;
    flex-direction: column;
    gap: 0.8rem;
  }
}
</style>

