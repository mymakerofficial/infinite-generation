<template>
  <div
    class="RenderContainer"
    ondragstart="return false;"
    ondrop="return false;"
  >
    <div :style="{ transform: `translate(${xOffset}px, ${yOffset}px)` }">
      <div v-for="y in gridHeight" :key="y">
        <div v-for="x in gridWidth" :key="x">
          <RenderSquare
            :x="(x - (gridWidth / 2 + 1) + gridXOffset) * tileWidth"
            :y="(y - (gridHeight / 2 + 1) + gridYOffset) * tileHeight"
            :width="tileWidth"
            :height="tileHeight"
          ></RenderSquare>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import RenderSquare from "./RenderTile.vue";
export default {
  name: "RenderContainer",
  components: { RenderSquare },

  data() {
    return {
      xOffset: 0,
      yOffset: 0,
      tileWidth: 100,
      tileHeight: 100,
      gridWidth: 20,
      gridHeight: 20,
    };
  },

  computed: {
    gridXOffset: function () {
      return -Math.floor(this.xOffset / this.tileWidth);
    },
    gridYOffset: function () {
      return -Math.floor(this.yOffset / this.tileHeight);
    },
  },

  created() {
    let mouseDown = false;
    let mouseDrag = false;
    let pointerStartOffset = { x: 0, y: 0 };
    let canvasStartOffset = { x: 0, y: 0 };

    document.addEventListener("mousedown", () => {
      mouseDrag = false;
      mouseDown = true;
    });

    document.addEventListener("mousemove", (e) => {
      if (mouseDown) {
        if (mouseDrag === false) {
          pointerStartOffset.x = e.x;
          pointerStartOffset.y = e.y;

          canvasStartOffset.x = this.xOffset;
          canvasStartOffset.y = this.yOffset;

          mouseDrag = true;
        }

        this.xOffset = canvasStartOffset.x + e.x - pointerStartOffset.x;
        this.yOffset = canvasStartOffset.y + e.y - pointerStartOffset.y;
      }
    });

    document.addEventListener("mouseup", () => {
      mouseDrag = false;
      mouseDown = false;
    });
  },
};
</script>

<style scoped></style>
