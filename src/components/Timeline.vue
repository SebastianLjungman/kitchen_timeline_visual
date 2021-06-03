<template>
  <div>
    <svg :viewBox="getViewBox">
     <rect v-for="(line, index) in lines" :x="line.start" y="0" :width="line.width" height="100000" :key="index" :class="line.class" @mouseover="showDetails"/>
    </svg>
  </div>
</template>

<script>
export default {
  name: 'Timeline',
  props: {
    data: Array,
    max: Number,
    offset: Number
  },
  computed: {
    lines() {
      const lines = []
      let start = null;
      let latestEnd = null;
      for (let item of this.data) {
        if (item.state === "on" && start === null) {
          start = item.time-this.offset;
        }
        else if (item.state === "off") {
          if (start < latestEnd) {
            start = latestEnd;
          }
          lines.push({start: start, width: item.time-start-this.offset, class: "normal"})
          start = null;
          latestEnd = item.time-this.offset;
        }
        else if (item.state !== "on") {
          lines.push({start: start, width: 43200, class: "error"});
          start = null
        }
      }
      return lines;
    },
    getViewBox() {
      return `0 0 ${this.max} 100000`;
    }
  },
  methods: {
    showDetails(e) {
      console.log(e.target.x.baseVal.value)
      this.$emit('info', {start: e.target.x.baseVal.value, duration: e.target.width.baseVal.value, pageX: e.pageX, pageY: e.pageY});
    }
  }
}
</script>

<style scoped>
svg {
  border-top: 1px dotted lightgray;
}
.normal {
  fill: black;
}
.error {
  fill: red;
}
</style>
