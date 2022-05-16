<template>
  <div
    class="RenderContainer"
    ondragstart="return false;"
    ondrop="return false;"
  >
    <div :style="{ transform: `translate(${xOffset}px, ${yOffset}px)` }">
      <div v-for="gridX in grid" :key="gridX">
        <div v-for="tile in gridX" :key="tile">
          <RenderSquare
            :x="tile.x - (gridWidth / 2) * tileWidth"
            :y="tile.y - (gridHeight / 2) * tileHeight"
            :width="tileWidth"
            :height="tileHeight"
            :renderScale="renderScale"
          ></RenderSquare>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import RenderSquare from "./RenderTile.vue";
import { Pane } from "tweakpane";

let BidirectionalModulo = function (a, b) {
  return ((a % b) + b) % b;
};

export default {
  name: "RenderContainer",
  components: { RenderSquare },

  data() {
    return {
      xOffset: 0,
      yOffset: 0,
      tileWidth: 68,
      tileHeight: 68,
      gridWidth: 13,
      gridHeight: 13,
      renderScale: 1,
      grid: [],
      rollX: 0,
      rollY: 0,
    };
  },

  computed: {
    gridXOffset() {
      return -Math.floor(this.xOffset / this.tileWidth);
    },
    gridYOffset() {
      return -Math.floor(this.yOffset / this.tileHeight);
    },
    totalWidth() {
      return this.tileWidth * this.gridWidth;
    },
    totalHeight() {
      return this.tileHeight * this.gridHeight;
    },
  },

  watch: {
    gridWidth() {
      this.create();
    },
    gridHeight() {
      this.create();
    },
    tileWidth() {
      this.flushGrid();
    },
    tileHeight() {
      this.flushGrid();
    },
    gridXOffset(newX, oldX) {
      this.rollOver(newX - oldX, 0);
    },
    gridYOffset(newY, oldY) {
      this.rollOver(0, newY - oldY);
    },
  },

  methods: {
    create() {
      // generate x
      this.grid = new Array(this.gridWidth);
      for (let x = 0; x < this.grid.length; x++) {
        // generate y
        this.grid[x] = new Array(this.gridHeight);
        for (let y = 0; y < this.grid[x].length; y++) {
          // fill with coordinates
          this.grid[x][y] = {
            x: x * this.tileWidth,
            y: y * this.tileHeight,
          };
        }
      }
    },
    flushGrid() {
      // clean up grid structure
      this.rollX = 0;
      this.rollY = 0;
      for (let x = 0; x < this.grid.length; x++) {
        for (let y = 0; y < this.grid[x].length; y++) {
          this.grid[x][y] = {
            x: (this.gridXOffset + x) * this.tileWidth,
            y: (this.gridYOffset + y) * this.tileHeight,
          };
        }
      }
    },
    rollOver(x, y) {
      for (let iX = 0; iX < Math.abs(x); iX++) {
        let dir = Math.sign(x);
        let left = +(dir === -1);
        let right = +(dir === 1);

        // move colum at rollX to front or back
        for (let i = 0; i < this.gridHeight; i++) {
          this.grid[this.rollX][i].x =
            (this.gridXOffset + (this.gridWidth - 1) * right + left) *
            this.tileWidth;
        }

        // iterate roll indicator
        this.rollX = BidirectionalModulo(this.rollX + dir, this.gridWidth);
      }

      for (let iY = 0; iY < Math.abs(y); iY++) {
        let dir = Math.sign(y);
        let up = +(dir === -1);
        let down = +(dir === 1);

        // move row at rollY to front or back
        for (let i = 0; i < this.gridWidth; i++) {
          this.grid[i][this.rollY].y =
            (this.gridYOffset + (this.gridHeight - 1) * down + up) *
            this.tileHeight;
        }

        // iterate roll indicator
        this.rollY = BidirectionalModulo(this.rollY + dir, this.gridHeight);
      }
    },
  },

  created() {
    this.create();

    let mouseDown = false;
    let mouseDrag = false;
    let pointerStartOffset = { x: 0, y: 0 };
    let canvasStartOffset = { x: 0, y: 0 };

    // -- register events --

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

    // -- debugging panel --

    const pane = new Pane();

    const fGrid = pane.addFolder({
      title: "Grid Controls",
      expanded: true,
    });

    fGrid.addInput(this, "tileWidth", { min: 1, max: 200, step: 1 });
    fGrid.addInput(this, "tileHeight", { min: 1, max: 200, step: 1 });
    fGrid.addSeparator();
    fGrid.addInput(this, "gridWidth", { min: 1, max: 50, step: 1 });
    fGrid.addInput(this, "gridHeight", { min: 1, max: 50, step: 1 });
    fGrid.addSeparator();
    fGrid
      .addButton({
        title: "flushGrid",
      })
      .on("click", () => {
        this.flushGrid();
      });

    const fGridInfo = pane.addFolder({
      title: "Grid Info",
      expanded: true,
    });

    fGridInfo.addMonitor(this, "xOffset", { interval: 100 });
    fGridInfo.addMonitor(this, "yOffset", { interval: 100 });
    fGridInfo.addSeparator();
    fGridInfo.addMonitor(this, "gridXOffset", { interval: 100 });
    fGridInfo.addMonitor(this, "gridYOffset", { interval: 100 });
    fGridInfo.addSeparator();
    fGridInfo.addMonitor(this, "rollX", { interval: 100 });
    fGridInfo.addMonitor(this, "rollY", { interval: 100 });

    const fRender = pane.addFolder({
      title: "Render",
      expanded: true,
    });

    fRender.addInput(this, "renderScale", { min: 0.01, max: 1, step: 0.01 });
  },
};
</script>

<style scoped></style>
