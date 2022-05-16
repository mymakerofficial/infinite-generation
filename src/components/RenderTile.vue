<template>
  <div
    :style="{
      transform: `translate(${x - width / 2}px, ${y - height / 2}px)`,
      width: `${width}px`,
      height: `${height}px`,
      /*background: `url('${imageUrl}') 0% 0% / cover`,*/
      /*transitionDuration: '.3s',*/
    }"
    style="position: absolute; top: 0; left: 0"
  >
    <div
      class="TileInfoText"
      :style="{
        fontSize: '9px',
      }"
    >
      x:{{ x }}<br />
      y:{{ y }}
    </div>
    <canvas
      ref="canvas"
      :width="width * renderScale"
      :height="height * renderScale"
      :style="{ width: `${width}px`, height: `${height}px` }"
      v-show="!loading"
    ></canvas>
  </div>
</template>

<script>
export default {
  name: "RenderSquare",

  props: ["x", "y", "width", "height", "renderScale"],

  data() {
    return {
      imageUrl: ``, //https://picsum.photos/seed/${this.x}x${this.y}/100/100
      loading: false,
    };
  },

  watch: {
    x() {
      this.generate();
    },
    y() {
      this.generate();
    },
    renderScale() {
      this.generate();
    },
  },

  methods: {
    generate() {
      this.loading = true;

      // timeout to reduce load
      setTimeout(() => {
        let canvas = this.$refs.canvas;
        let ctx = canvas.getContext("2d");

        const imageData = ctx.createImageData(canvas.width, canvas.height);

        // Iterate through every pixel
        for (let i = 0; i < imageData.data.length; i += 4) {
          // calculate pixel index
          let p = i / 4;
          // calculate relative pixel coordinate
          let x = p % canvas.width;
          let y = Math.round(p / canvas.height);
          // calculate absolute pixel coordinate, relative to complete image
          let aX = x + this.x;
          let aY = y + this.y;

          // Modify pixel data
          imageData.data[i] =
            ((Math.cos((aY / (canvas.height * 4)) * this.renderScale) + 1) /
              2) *
            255; // R value
          imageData.data[i + 1] =
            ((Math.sin((aX / (canvas.width * 4)) * this.renderScale) + 1) / 2) *
            255; // G value
          imageData.data[i + 2] =
            ((Math.sin((aY / (canvas.height * 4)) * this.renderScale) + 1) /
              2) *
            255; // B value
          imageData.data[i + 3] = 255; // A value
        }

        // Draw image data to the canvas
        ctx.putImageData(imageData, 0, 0);

        this.loading = false;
      }, Math.random() * 100);
    },
  },

  mounted() {
    this.generate();
  },
};
</script>

<style scoped></style>
