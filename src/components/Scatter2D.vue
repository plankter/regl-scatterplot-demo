<template>
  <canvas ref="canvas" :width="width" :height="height" />
</template>

<script lang="ts">
import createScatterplot, { createTextureFromUrl, createRegl } from "regl-scatterplot";
import { Component, Prop, Vue, Watch } from "vue-property-decorator";

@Component
export default class Scatter2D extends Vue {

  @Prop(String) title;
  @Prop(Number) width;
  @Prop(Number) height;

  points: any[] = [];
  scatterplot: any;
  selection: any[] = [];

  pointoverHandler(pointId) {
    const [x, y, category, value] = this.points[pointId];
    console.log(`X: ${x}\nY: ${y}\nCategory: ${category}\nValue: ${value}`);
  }

  pointoutHandler(pointId) {
    const [x, y, category, value] = this.points[pointId];
    console.log(`X: ${x}\nY: ${y}\nCategory: ${category}\nValue: ${value}`);
  }

  selectHandler({ points: selectedPoints }) {
    console.log("Selected:", selectedPoints);
    this.selection = selectedPoints;
  }

  deselectHandler() {
    console.log("Deselected:", this.selection);
    this.selection = [];
  }

  resizeHandler() {
    const canvas = this.$refs.canvas as any;
    if (canvas) {
      const rect = canvas.getBoundingClientRect();
      if (rect) {
        this.scatterplot.set({ width: rect.width, height: rect.height });
      }
    }
  }

  async mounted() {
    const canvas = this.$refs.canvas as any;

    const { width, height } = canvas.getBoundingClientRect();

    const regl = createRegl(canvas);
    const backgroundImage = await createTextureFromUrl(
      regl,
      `https://picsum.photos/${Math.min(640, width)}/${Math.min(640, height)}/?random`,
      true
    );

    this.scatterplot = createScatterplot({
      canvas,
      width,
      height,
      opacity: 1,
      pointSize: 2,
      pointSizeSelected: 1,
      pointOutlineWidth: 1,
      lassoMinDelay: 15,
      backgroundImage: backgroundImage,
    });

    this.scatterplot.subscribe("pointover", this.pointoverHandler);
    this.scatterplot.subscribe("pointout", this.pointoutHandler);
    this.scatterplot.subscribe("select", this.selectHandler);
    this.scatterplot.subscribe("deselect", this.deselectHandler);

    window.addEventListener("resize", this.resizeHandler);

    this.points = new Array(1000).fill(0).map(() => [
      -1 + Math.random() * 2, // x
      -1 + Math.random() * 2, // y
      Math.round(Math.random()), // category
      Math.random() // value
    ])

    this.scatterplot.draw(this.points);
  }

  beforeDestroy() {
    this.scatterplot.destroy();
  }
}
</script>
