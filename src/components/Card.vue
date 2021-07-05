<template>
  <div class="card" :class="flippedStyles" @click="selectCard">
    <div class="card-face is-front">
      <img
        :src="`/images/image-${value}.jpg`"
        :alt="value"
        class="image-card"
      />
      <img v-if="matched" src="/images/check-mark.svg" class="icon-check" />
    </div>
    <div class="card-face is-back">
      <img src="/images/image-back.jpg" />
    </div>
  </div>
</template>

<script>
import { computed } from "vue";

export default {
  name: "Card",
  props: {
    matched: {
      type: Boolean,
      default: false,
    },
    position: {
      type: Number,
      required: true,
    },
    value: {
      type: String,
      required: true,
    },
    visible: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, context) {
    const flippedStyles = computed(() => {
      if (props.visible) {
        return "is-flipped";
      } else return "none";
    });

    const selectCard = () => {
      context.emit("select-card", {
        position: props.position,
        faceValue: props.value,
      });
    };

    return {
      flippedStyles,
      selectCard,
    };
  },
};
</script>

<style lang="css" scoped>
.card {
  position: relative;
  transition: 0.5s transform ease-in;
  transform-style: preserve-3d;
}

.card.is-flipped {
  transform: rotateY(180deg);
}

.card-face {
  width: 100%;
  height: 100%;
  position: absolute;
  border-radius: 10px;
  cursor: pointer;
  backface-visibility: hidden;
}

.card-face.is-front {
  width: 100%;
  height: 100%;
  object-fit: contain;
  background-image: url(/images/image-1.jpg);
  color: white;
  transform: rotateY(180deg);
}

.image-card {
  height: 100%;
  width: 100%;
  border-radius: 10px;
}

.icon-check {
  width: 30px;
  height: 30px;
  position: absolute;
  right: 0;
  bottom: 0;
}

.card-face.is-back img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 10px;
}
</style>
