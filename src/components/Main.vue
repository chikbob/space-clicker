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
.page__container {
  background-color: #1e1e2f; /* Темний фон */
  color: #ffffff; /* Білий текст */
  padding: 10px 6px;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.page__header {
  text-align: center;
  margin-bottom: 10px;
}

.page__title {
  font-size: 3rem;
  font-weight: bold;
  margin-bottom: 20px;
}

.page__score {
  font-size: 2rem;
  font-weight: 600;
  margin-bottom: 16px;
}

.page__button {
  background-color: #9b4dca; /* Фіолетовий */
  color: #ffffff; /* Білий текст */
  padding: 20px 40px;
  border-radius: 20px;
  font-size: 1.5rem;
  font-weight: bold;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transition: background-color 0.3s ease;

  &:hover {
    background-color: #6a2c9f; /* Фіолетовий при наведенні */
  }
}

.page__content {
  display: flex;
  flex-direction: row;
  align-items: start;
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto;

  @media(max-width: 768px) {
    flex-direction: column;
    align-items: center;
    justify-content: space-between;
  }
}

.page__section {
  flex: 1;
  margin: 50px 0 0 0;
  width: 600px; /* Ширина по замовчуванню */
  padding: 20px 40px;
  background-color: #33333d; /* Темно-сірий */
  border-radius: 16px;

  @media(max-width: 768px) {
    width: 100%; /* Ширина 100% для телефонів */
    padding: 10px; /* Зменшення відступів на маленьких екранах */
  }
}

.page__section-title {
  font-size: 2rem;
  font-weight: 600;
  border-bottom: 2px solid;
  padding-bottom: 10px;
  margin-bottom: 20px;
  text-align: center;

  &--active {
    color: #9b4dca; /* Фіолетовий */
    border-color: #9b4dca; /* Фіолетовий */
  }

  &--passive {
    color: #3b8bbd; /* Синій */
    border-color: #3b8bbd; /* Синій */
  }
}

.page__upgrades {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

// Стилі для компонента UpgradeCard
.upgrade-card {
  background-color: #33333d; /* Темно-сірий */
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;

  &__title {
    font-size: 1.25rem;
    font-weight: bold;
    margin-bottom: 10px;
  }

  &__description {
    font-size: 1rem;
    color: #a9a9a9; /* Світло-сірий */
    margin-bottom: 16px;
  }

  &__button {
    background-color: #4caf50; /* Зелений */
    color: #ffffff; /* Білий текст */
    padding: 12px;
    border-radius: 8px;
    width: 100%;
    transition: background-color 0.3s ease;

    &:disabled {
      background-color: #b0b0b0; /* Сірий для вимкнених */
    }

    &:hover {
      background-color: #45a049; /* Зелений при наведенні */
    }
  }
}
</style>

