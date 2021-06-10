
<template>
  <div>
    <div id="app">
      <VueTreeCarousel
        ref="treecarousel"
        :slides="slides"
        :threshold="100"
        :arrows="true"
        v-slot="{ slide }"
      >
        <iframe
          v-if="isVideo(slide.content)"
          class="fullsize"
          :src="slide.content"
          title="YouTube video player"
          frameborder="0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
        <div v-else class="fullsize" :style="{ background: slide.background }">
          <div class="text">
            <vue-markdown>{{ slide.content }}</vue-markdown>
          </div>
        </div>
      </VueTreeCarousel>
    </div>
  </div>
</template>

<script>
import VueTreeCarousel from "./../src/VueTreeCarousel.vue";
import VueMarkdown from "@adapttive/vue-markdown";

export default {
  name: "App",
  components: { VueTreeCarousel, VueMarkdown },
  methods: {
    isVideo(content) {
      console.log(content);
      return (
        typeof content === "string" &&
        content.startsWith("https://www.youtube.com/embed")
      );
    },
  },
  data() {
    const getLoremPicsum = (id) => `center / cover no-repeat url(https://picsum.photos/id/${id}/1200/800)`;
    const generateTree = (deep = 3, length = 4, prefix = "", fn) =>
      Array.from({ length }, (_, i) => {
        const content = prefix + (i + 1);
        return {
          content,
          background: fn ? fn(i, deep, content) : undefined,
          children:
            deep > 0 ? generateTree(deep - 1, length, content, fn) : undefined,
        };
      });

    return {
      slides: [
        {
          content:
            "# Tree Carousel\n- Keyboard navigation\n  - arrow keys & wasd\n- Button navigation\n- Swipe gestures",
          background: "#a8e6cf",
        },
        {
          content: "## Navigate a tree",
          background: "#dcedc1",
          children: generateTree(2, 4, "", (a, b, content) => {
            return getLoremPicsum(content);
          }),
        },
        {
          content:
            "## Video\n- Loses swipe and keyboard navigation when interacted with\n- Use arrow navigation as fallback",
          background: "#ffd3b6",
          children: [
            {
              content: "https://www.youtube.com/embed/kH6D2NtSKKo?start=24",
            },
            {
              content: "https://www.youtube.com/embed/__ffgI9AMoA",
            },
          ],
        },
        {
          content:
            "## Todo\n- Responsive\n- 🐛 Fixing 🐛\n- Refactor\n- Deploy",
          background: "#ffaaa5",
        },
        {
          content: "## Fin.",
          background: "#ff8b94",
        },
      ],
      index: { x: 0, y: 0 },
    };
  },
};
</script>

<style>
html,
body {
  width: 100vw;
  height: 100vh;
  margin: 0;
  overflow: hidden;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.fullsize {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.text {
  font-family: "Courier New", Courier, monospace;
  font-size: 3em;
}
</style>
