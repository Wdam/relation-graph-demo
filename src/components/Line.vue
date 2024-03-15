<template>
  <g @click="onLineClick(line, $event)">
    <!-- 定义箭头 -->
    <defs>
      <marker id="arrowhead" markerWidth="10" markerHeight="7"
              refX="0" refY="3.5" orient="auto">
        <path d="M0,0 L10,3.5 L0,7 Z" fill="rgba(153, 77, 22, 1)" />
      </marker>
    </defs>
    <!-- 使用箭头的路径 -->
    <path :d="pathData"
          stroke="rgba(153, 77, 22, 1)"
          :style="{'stroke-width': 1 + 'px'}"
          fill="none"
          marker-end="url(#arrowhead)" /> <!-- 引用箭头 -->

    <!-- 文字说明 -->
    <text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">
      Your text here
    </text>

    <g ref="flower" />
  </g>
</template>

<script>
export default {
  name: 'MyLine',
  components: { },
  props: {
    relationGraph: { type: Object, default: () => ({}) },
    link: { type: Object, default: () => ({}) },
    relationIndex: { type: Object, default: () => ({}) },
    line: { type: Object, default: () => ({}) }
  },
  computed: {
    pathData() {
      const fx = this.link.fromNode.x + 50;
      const fy = this.link.fromNode.y + 50;
      const tx = this.link.toNode.x + 50;
      const ty = this.link.toNode.y + 50;
      return 'M ' + fx + ' ' + fy + ' L ' + tx + ' ' + ty;
    }
  },
  data() {
    return {
      position: null
    };
  },
  mounted() {
    this.task = setInterval(() => {
      this.updateFlower();
    }, 500);
  },
  beforeDestroy() {
    clearInterval(this.task);
  },
  methods: {
    getPositon() {
      const fx = this.link.fromNode.x + 50;
      const fy = this.link.fromNode.y + 50;
      const tx = this.link.toNode.x + 50;
      const ty = this.link.toNode.y + 50;
      const x = fx + (tx - fx) / 2;
      const y = fy + (ty - fy) / 2;
      return { x, y };
    },
    updateFlower() {
      const position = this.getPositon();
      if (
          this.position &&
          this.position.x === position.x &&
          this.position.y === position.y
      ) {
        return;
      }
      this.position = position;
      // this.createFlower();
    },
    onLineClick(lineObject, linkObject, $event) {
      console.log('onLineClick:', lineObject);
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
