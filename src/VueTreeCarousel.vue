<template>
  <div
    class="slide"
    v-hammer:pan="updateOffset"
    v-hammer:panend="doSwipe"
    :style="{
      transform: `translate(${offset.x}px, ${offset.y}px)`,
      transition: transitioning && 'transform 0.5s ease-in',
    }"
  >
    <tree ref="tree" :slides="slides" v-slot="{ slide }">
      <slot v-bind:slide="slide"></slot>
      <div
        v-if="arrows"
        :class="{
          arrows: true,
          hide: hideArrows || transitioning,
          'hide-transition': !hideArrows && !transitioning,
        }"
      >
        <div v-if="hasUpward" class="arrow up" @click="goUpward"></div>
        <div v-if="hasBackward" class="arrow left" @click="goBackward"></div>
        <div v-if="hasForward" class="arrow right" @click="goForward"></div>
        <div v-if="hasDownward" class="arrow down" @click="goDownward"></div>
      </div>
    </tree>
  </div>
</template>

<script>
import tree from "./Tree";

export default {
  name: "treecarousel",
  components: { tree },
  props: {
    slides: {
      type: Array,
      default() {
        return [];
      },
    },
    threshold: {
      type: Number,
      default: 50,
    },
    arrows: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      hasForward: false,
      hasBackward: false,
      hasDownward: false,
      hasUpward: false,
      direction: null,
      transitioning: false,
      hideArrows: false,
      transitionInterval: null, // kludge
      offset: { x: 0, y: 0 },
    };
  },
  methods: {
    updateOffset(evt) {
      this.hideArrows = true;
      if (!this.direction) this.direction = evt.additionalEvent;

      if (this.direction === "panleft" || this.direction === "panright")
        this.offset.x = evt.deltaX;
      else if (this.direction === "pandown" || this.direction === "panup")
        this.offset.y = evt.deltaY;
    },
    doSwipe() {
      if (this.offset.x < -this.threshold) {
        this.goForward();
      } else if (this.offset.x > this.threshold) {
        this.goBackward();
      } else if (this.offset.y < -this.threshold) {
        this.goDownward();
      } else if (this.offset.y > this.threshold) {
        this.goUpward();
      } //
      this.direction = null;
      this.offset = { x: 0, y: 0 };
      this.hideArrows = false;
    },
    setTransition() {
      this.transitioning = true;
      if (this.transitionInterval) clearTimeout(this.transitionInterval); // kludge
      this.transitionInterval = setTimeout(
        () => (this.transitioning = false),
        500
      ); // kludge
    },
    goForward() {
      this.$refs.tree.goForward();
      this.setArrows();
      this.setTransition();
    },
    goBackward() {
      this.$refs.tree.goBackward();
      this.setArrows();
      this.setTransition();
    },
    goDownward() {
      this.$refs.tree.goDownward();
      this.setArrows();
      this.setTransition();
    },
    goUpward() {
      this.$refs.tree.goUpward();
      this.setArrows();
      this.setTransition();
    },
    setArrows() {
      setTimeout(() => {
        this.hasForward = this.$refs.tree?.hasForward();
        this.hasBackward = this.$refs.tree?.hasBackward();
        this.hasDownward = this.$refs.tree?.hasDownward();
        this.hasUpward = this.$refs.tree?.hasUpward();
      }, 1);
    },
    onKeyPress(evt) {
      switch (evt.keyCode) {
        case 65:
        case 37:
          this.goBackward();
          break;
        case 87:
        case 38:
          this.goUpward();
          break;
        case 68:
        case 39:
          this.goForward();
          break;
        case 83:
        case 40:
          this.goDownward();
          break;
      }
    },
  },
  mounted() {
    document.addEventListener("keydown", (e) => this.onKeyPress(e));
    this.setArrows();
  },
};
</script>

<style scoped>
.slide {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

.arrows {
  opacity: 1;
}

.hide-transition {
  transition: opacity 0.5s ease-in;
}

.hide {
  opacity: 0;
  transition: opacity 0;
}

.arrow {
  border: solid black;
  border-width: 0 8px 8px 0;
  border-radius: 4px;
  display: inline-block;
  padding: 8px;
  width: 64px;
  height: 64px;
  position: absolute;
}

.arrow:hover {
  border: solid black;
  border-width: 8px;
}

.right {
  transform: rotate(-45deg) translate(50%, -50%);
  right: 32px;
  top: 50%;
}

.left {
  transform: rotate(135deg) translate(-50%, 50%);
  left: 32px;
  top: 50%;
}

.up {
  transform: rotate(-135deg) translate(50%, -50%);
  top: 32px;
}

.down {
  transform: rotate(45deg) translate(-50%, 50%);
  bottom: 32px;
}
</style>
