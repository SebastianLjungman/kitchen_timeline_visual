<template>
  <div>
    <svg :viewBox="getViewBox">
     <rect v-for="(line, index) in lines" :x="line.start" y="0" :width="line.width" height="100000" :key="index" :class="line.class" :alt="getDate(line.start) + ' - ' + getDuration(line.width)" @mouseover="showDetails"/>
    </svg>
  </div>
</template>

<script>
export default {
  name: 'Timeline',
  props: {
    data: Array,
    offset: Number,
    max: Number
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
          if (start + this.offset < latestEnd) {
            start = latestEnd;
          }
          lines.push({start: start, width: item.time-start-this.offset, class: "normal"})
          start = null;
          latestEnd = item.time;
        }
        else if (item.state !== "on") {
          lines.push({start: start, width: 43200, class: "error"});
          start = null
        }
      }
      return lines;
    },
    getViewBox() {
      return `0 0 2000000 100000`;
    }
  },
  methods: {
    getDate(time) {
      const date = new Date((time+this.offset) * 1000)
      return `${date.getDate()}/${date.getMonth()} ${date.getHours()}:${date.getMinutes()}`
    },
    getDuration(time) {
      return `${Math.floor(time/3600)} h, ${time%60} min`
    },
    showDetails(e) {
      this.$emit('info', e);
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
