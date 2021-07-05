<template>
  <h1>Peek-a-Vue</h1>
  <section class="description">
    <p>Welcome to Peek-a-Vue!</p>
    <p>A card matching game powered by Vue.js 3!</p>
  </section>
  <transition-group tag="section" class="game-board" name="shuffle-card">
    <Card
      v-for="card in cardList"
      :key="`${card.value}-${card.variant}`"
      :matched="card.matched"
      :value="card.value"
      :visible="card.visible"
      :position="card.position"
      @select-card="flipCard"
    />
  </transition-group>
  <h2>{{ status }}</h2>
  <button v-if="newPlayer" @click="startGame" class="button">Start Game</button>
  <button v-else @click="restartGame" class="button">Restart Game</button>
</template>

<script>
import _ from "lodash";
import { computed, ref, watch } from "vue";
import { launchConfetti } from "./utilities/confetti";
import Card from "./components/Card.vue";

export default {
  name: "App",
  components: {
    Card,
  },
  setup() {
    const cardList = ref([]);
    const userSelection = ref([]);
    const newPlayer = ref(true)

    const startGame = () => {
      newPlayer.value = false;

      restartGame();
    }

    const status = computed(() => {
      if (remainingPairs.value === 0) {
        return "Player wins!";
      } else {
        return `Remaining Pairs: ${remainingPairs.value}`;
      }
    });

    const remainingPairs = computed(() => {
      const remainingCards = cardList.value.filter(
        (card) => card.matched === false
      ).length;

      return remainingCards / 2;
    });

    const restartGame = () => {
      cardList.value = _.shuffle(cardList.value);

      cardList.value = cardList.value.map((card, index) => {
        return {
          ...card,
          matched: false,
          position: index,
          visible: false,
        };
      });
    };

    const cardItems = [1, 2, 3, 4, 5, 6, 7, 8];

    cardItems.forEach((item) => {
      cardList.value.push({
        value: item,
        variant: 1,
        visible: false,
        position: null,
        matched: false,
      });

      cardList.value.push({
        value: item,
        variant: 2,
        visible: true,
        position: null,
        matched: false,
      });
    });

    cardList.value = cardList.value.map((card, index) => {
      return {
        ...card,
        position: index,
      };
    });

    const flipCard = (payload) => {
      cardList.value[payload.position].visible = true;

      if (userSelection.value[0]) {
        if (
          userSelection.value[0].position === payload.position &&
          userSelection.value[0].faceValue === payload.faceValue
        ) {
          return;
        } else {
          userSelection.value[1] = payload;
        }
      } else {
        userSelection.value[0] = payload;
      }
    };

    watch(remainingPairs, (currentValue) => {
      if (currentValue === 0) {
        launchConfetti();
      }
    });

    watch(
      userSelection,
      (currentValue) => {
        if (currentValue.length == 2) {
          const cardOne = currentValue[0];
          const cardTwo = currentValue[1];

          if (cardOne.faceValue === cardTwo.faceValue) {
            cardList.value[cardOne.position].matched = true;
            cardList.value[cardTwo.position].matched = true;
          } else {
            setTimeout(() => {
              cardList.value[cardOne.position].visible = false;
              cardList.value[cardTwo.position].visible = false;
            }, 2000);
          }

          userSelection.value.length = 0;
        }
      },
      { deep: true }
    );

    return {
      cardList,
      flipCard,
      userSelection,
      status,
      restartGame,
      startGame,
      newPlayer
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #4f143f;
  background-image: url("/images/image-bg.png");
  height: 100vh;
}

h1,
h2,
p,
button {
  margin: 0;
}

h1 {
  padding: 30px 0 10px 0;
}

h2 {
  padding: 20px 0 10px 0;
}

body {
  margin: 0;
}

.description {
  margin-bottom: 20px;
}

.game-board {
  display: grid;
  grid-template-columns: repeat(4, 120px);
  grid-template-rows: repeat(4, 120px);
  grid-column-gap: 10px;
  grid-row-gap: 10px;
  justify-content: center;
}

.shuffle-card-move {
  transition: transform 0.8s ease-in;
}

.button {
  background-color: #ffaba2;
  border: none;
  color: inherit;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
  box-shadow: 0 1px 0 0 #ffbf87, -1px 2px 0 0 #ffbf87, -2px 3px 0 0 #ffbf87,
    -3px 4px 0 0 #ffbf87;
}

.button:hover {
  background-color: #ffda72;
}

.button:active {
  transform: translate(-3px, 4px);
  box-shadow: none;
}
</style>
