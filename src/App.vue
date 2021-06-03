<template>
  <div>
    <div class="info" :style="{left: infoLocation.left+'px' , top: infoLocation.top+'px'}">
      {{info}}
    </div>
    <div class="wrapper" v-for="(machine, mkey)  in machines" :key="'m'+mkey">
      <div>
        {{machine}}
      </div>
        <Timeline :data="getMachineSeries(machine)" :offset="Number.parseInt(offset)" :max="Number.parseInt(max)" @info="setInfo"/>
    </div>
    <legend> start <input type="range" :min="offsetInit" :max="offsetMax" v-model="offset"></legend>
    <legend>zoom <input type="range" min="10000" max="2000000" v-model="max"></legend>
  </div>
</template>
<script>

import Timeline from "@/components/Timeline.vue";

export default {
  components: {
    Timeline
  },
  data () {
    return {
      data: [],
      machines: ['frima1', 'frying1', 'frying2', 'boiler1', 'boiler2', 'boiler3', 'oven1', 'oven2', 'oven3', 'oven4', 'oven5', 'oven6', 'ovenS1', 'stoveS1', 'roughDish1', 'roughDish2', 'tunnelDish1', 'serving1'],
      offsetInit: 1620270000,
      offsetMax: 1623000000,
      offset: 1620270000,
      max: 2000000,
      info: "",
      infoLocation: {left: 0, top: 0}
    }
  },
  created () {
    this.fetchData();
  },
  methods: {
    fetchData () {
      fetch("http://user.it.uu.se/~mikla253/sebastian/storedata.php")
      .then((response) => {
        if (response.ok) {
          return response.json();
        }
        throw response;
      })
      .then((json) => {
        this.data = json.data;
      })
      .catch((err) => {
        if (err.message) {
          this.setError(err.message);
        }
      });
    },
    setError(msg) {
      console.log(msg);
    },
    getMachineSeries(machine) {
      const ret = []
      for(let item of this.data) {
        if (item.machine === machine) {
          ret.push(item)
        }
      }
      return ret;
    },
    setInfo(d) {
      this.info= `${this.getDate(d.start)} ${this.getDuration(d.duration)}`
      this.infoLocation = {left: d.pageX, top: d.pageY-5}
    },
    getDate(time) {
      const date = new Date((time + Number.parseInt(this.offset)) * 1000)
      return `${date.getDate()}/${date.getMonth()+1} ${date.getHours()}:${date.getMinutes()}`
    },
    getDuration(time) {
      return `${Math.floor(time/3600)} h, ${Math.floor(time%3600/60)} min`
    },
  }
}
</script>
<style>
  body {
    font-family: sans-serif;
    font-size: 10pt;
  }
  input[type="range"] {
    width:1000px;
  }
  .wrapper {
    display: grid;
    grid-template-columns: 80px auto;
    grid-template-rows: repeat(auto-fill, 20px);
    overflow: hidden;
  }
  .info {
    position: absolute;
    background-color:rgba(255,255,255,0.8);
  }
</style>