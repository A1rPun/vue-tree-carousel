<template>
  <div v-frag>
    <div
      class="slide"
      :style="{ transform: `translate(${x * 100}%, ${y * 100}%)` }"
    >
      <div
        v-for="(slide, index) in slides"
        :key="slide.content"
        class="slide"
        :style="{ transform: `translate(${index * 100}%)` }"
      >
        <slot v-bind:slide="slide"></slot>
      </div>
    </div>
    <tree
      v-if="y <= 0 && slides[currentSlide].children"
      ref="downward"
      :slides="slides[currentSlide].children"
      :verticalIndex="1"
      @downward="_goDownward()"
      v-slot="{ slide }"
    >
      <slot v-bind:slide="slide"></slot>
    </tree>
  </div>
</template>

<script>
import frag from "vue-frag";

export default {
  name: "tree",
  directives: {
    frag,
  },
  props: {
    slides: {
      type: Array,
      default() {
        return [];
      },
    },
    verticalIndex: {
      type: Number,
      default: 0,
    },
    buttons: {
      type: Boolean,
      default: false,
    },
  },
  watch: {
    slides() {
      this.x = 0;
    },
  },
  computed: {
    currentSlide() {
      return this.slides.length - (this.slides.length + this.x);
    },
    canForward() {
      return this.x > -this.slides.length + 1;
    },
    canDownward() {
      return this.y || this.$refs.downward;
    },
  },
  data() {
    return {
      transitioning: false,
      x: 0,
      y: 0,
    };
  },
  methods: {
    hasForward() {
      if (this.y) {
        return this.$refs.downward?.hasForward();
      } else {
        return this.canForward;
      }
    },
    hasBackward() {
      if (this.y) {
        return this.$refs.downward?.hasBackward();
      } else {
        return this.x < 0;
      }
    },
    hasDownward() {
      if (this.$refs.downward) {
        return this.$refs.downward.hasDownward();
      } else {
        return (
          this.y > 0 ||
          (!this.y && this.slides[this.currentSlide].children?.length)
        );
      }
    },
    hasUpward() {
      return this.y < 0;
    },
    goForward() {
      if (this.transitioning) return;

      if (this.y) {
        this.$refs.downward?.goForward();
      } else {
        this._goForward();
      }
    },
    _goForward() {
      if (!this.canForward) return;
      this.x -= 1;
      this.setTransition();
    },
    goBackward() {
      if (this.transitioning) return;

      if (this.y) {
        this.$refs.downward?.goBackward();
      } else {
        this._goBackward();
      }
    },
    _goBackward() {
      if (!this.x) return;
      this.x += 1;
      this.setTransition();
    },
    goDownward() {
      if (this.transitioning) return;

      if (this.$refs.downward) {
        this.$refs.downward.goDownward();
      } else {
        this._goDownward();
      }
    },
    _goDownward() {
      if (!this.canDownward) return;

      this.y -= 1;
      this.setTransition();
      this.$emit("downward");
    },
    goUpward() {
      if (this.transitioning) return;
      if (this.y >= 0) return;

      this._goUpward();
    },
    _goUpward() {
      this.y += 1;
      this.setTransition();
      this.$refs.downward?._goUpward();
    },
    setTransition() {
      this.transitioning = true;
      setTimeout(() => (this.transitioning = false), 500); // kludge
    },
  },
  mounted() {
    this.y = this.verticalIndex;
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
  transition: transform 0.5s ease-in;
}
</style>
