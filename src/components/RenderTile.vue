<template>
  <div :style="{ transform: `translate(${x}px, ${y}px)`, width: `${width}px`, height: `${height}px`, background: `url('https://picsum.photos/seed/${x}x${y}/100/100') 0% 0% / cover` }" style="position: absolute;top: 0; left: 0;">
    <!--<canvas ref="canvas" :width="`${this.width}px`" :height="`${this.height}px`" :style="{ width: `${width}px`, height: `${height}px` }"></canvas>-->
  </div>
</template>

<script>
export default {
  name: "RenderSquare",

  props: ["x", "y", "width", "height"],

  methods: {
    generate: function () {},
  },

  created() {
    this.generate();
  },
};
</script>

<style scoped></style>
