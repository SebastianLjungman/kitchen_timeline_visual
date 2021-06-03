<template>
  <div>
    <div class="info" :style="{left: infoLocation.left+'px' , top: infoLocation.top+'px'}">
      {{info}}
    </div>
    <div class="wrapper" v-for="(machine, mkey)  in machines" :key="'m'+mkey">
      <div>
        {{machine}}
      </div>
        <Timeline :data="getMachineSeries(machine)" :offset="offset" :max="max" @info="setInfo"/>
    </div>
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
      offset: 1620270000,
      max: 1621000000,
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
        this.data = json.data
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
    setInfo(e) {
      this.info= e.target.attributes.alt.value;
      this.infoLocation = {left: e.pageX, top: e.pageY-5}
    }
  }
}
</script>
<style>
  body {
    font-family: sans-serif;
    font-size: 10pt;
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