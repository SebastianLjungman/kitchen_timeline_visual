<template>
  <div id="viewBoxWrapper">
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
          //Extracts class by removing last character (number). Assumes less than 10 appliances!
          let machineClass = item.machine.slice(0, -1);

          lines.push({start: start, width: item.time-start-this.offset, class: [item.machine, machineClass]})
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
      console.log(e.target.classList[0]);
      this.$emit('info', {start: e.target.x.baseVal.value, duration: e.target.width.baseVal.value, pageX: e.pageX, pageY: e.pageY, machineName: e.target.classList[0]});
    }
  }
}
</script>

<style scoped>

#viewBoxWrapper {
  margin-left: 40px;
}

svg {
  border-top: 1px dotted lightgray;
}
.normal {
  fill: black;
}
.error {
  fill: rgb(255, 0, 200);
}



.frima {
  fill: #FF516C;
}

.frying {
  fill: #FBCC3C;
}

.boiler {
  fill: #00E0FF;
}

.oven {
  fill: #BC74FF;
}

.ovenS {
  fill: #219653;
}

.stoveS {
  fill: #6FCF97;
}

.roughDish {
  fill: #079dcfea;
}

.tunnelDish {
  fill: #079dcfea;
}

.serving {
  fill: #ff0000;
}

</style>
