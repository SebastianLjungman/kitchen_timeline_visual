<template>
  <div id="app">
    <div class="info" :style="{left: infoLocation.left+'px' , top: infoLocation.top+'px'}">
      {{info}}
    </div>
    <div id="legendContainer">
      <legend>Start <input type="range" :min="offsetInit" :max="offsetMax2" v-model="offset"></legend>
      <legend>Zoom<input type="range" min="200000" max="3000000" v-model="max"></legend>
      <legend>Graph Height<input id="slider" type="range" min="590" max="2000" v-model="graphHeight"></legend>
    </div>
    <Foodstats/>
      <!-- <input type="button" @click="changeGrafanaURL"> -->
    <div class="wrapper" v-for="(machine, mkey)  in machines" :key="'m'+mkey">
      <div id="machineNamesList">
        {{machine}}
      </div>
        <Timeline :data="getMachineSeries(machine)" :offset="Number.parseInt(offset)" :max="Number.parseInt(max)" @info="setInfo"/>
    </div>
    <div id="gd">
    <iframe id= "grafana" :src="grafanaStart" frameborder="0" :height="graphHeight"></iframe>
    </div> 
  </div>
  
</template>
<script>

import Timeline from "@/components/Timeline.vue";
import axios from "axios";
import Foodstats from './components/Foodstats.vue';

export default {
  components: {
    Timeline, Foodstats
  },
  data () {
    return {
      data: [],
      machines: null,
      offsetInit: 1620270000,
      offsetMax: 1623000000,
      offset: 1620270000,
      max: 1972000,
      info: "",
      infoLocation: {left: 0, top: 0},

      //Sebastian's additions: previous grafana from value: 1620275400000, to value: 1622270100 
      graphHeight: 900,
      grafanaSrc: "https://view.stuns.i0t.se/grafana/d-solo/UV-lugCGz/skolkok-effekt-med-nivaer?orgId=3&from=",
      grafanaSrcDomarringen: "https://view.stuns.i0t.se/grafana/d-solo/UV-lugCGz/skolkok-effekt-med-nivaer?orgId=3&from=",
      validKitchenNames: ["tiunda", "domarringen", "stenhagen"],
      kitchenName: null
    }
  },
  created () {
    let uri = window.location.search.substring(1); 
    let params = new URLSearchParams(uri);
    this.kitchenName = params.get("kitchen_name");
    console.log(this.kitchenName);

    if(this.kitchenName === "tiunda" ) {
      this.machines = ['frying1', 'frying2', 'frima1', 'boiler1', 'boiler2', 'boiler3', 'oven1', 'oven2', 'oven3', 'oven4', 'oven5', 'oven6', 
      'ovenS1', 'stoveS1', 'roughDish1', 'roughDish2', 'tunnelDish1', 'serving1'];
    }

    else if(this.kitchenName === "domarringen") {
      this.machines = ['boilerS1Dom', 'ovenS1Dom', 'oven1Dom', 'oven2Dom', 'oven3Dom', 'frying1Dom', 'stove1Dom', 
      'boiler1Dom', 'boiler2Dom', 'boiler3Dom', 'roughDish1Dom', 'tunnelDish1Dom'];
    }

    this.fetchData();
  },
    mounted() {
    //Adress måste innehålla giltigt värde på kitchen_name (se validKitchenNames), t.ex.
    //http://localhost/visualization_new_school/index.html?kitchen_name=domarringen

    if(this.validKitchenNames.includes(this.kitchenName)) {
      // alert("Köket existerar!");
    }
    else {
      document.getElementById("app").style.display = "none";
      alert("Köket finns inte med i listan! Kontrollera att rätt adress har angivits!");
    }
  },
  computed: {
    grafanaStart: function(){
      if(this.kitchenName === "domarringen") {
      return this.grafanaSrcDomarringen + (this.offset * 1000) + "&to=" + (this.offset * 1000 + 2000100000) + "&theme=light&panelId=10"
      }
      else {
        return this.grafanaSrc + (this.offset * 1000) + "&to=" + (this.offset * 1000 + 2000100000) + "&theme=light&panelId=9"
        //https://view.stuns.i0t.se/grafana/d-solo/UV-lugCGz/skolkok-effekt-med-nivaer?orgId=3&from=1630038567802&to=1630060167802&theme=light&panelId=10
      }
    },
    //Bör funka, även om nya värden inte verkar dyka upp korrekt (FELSÖK!)
    offsetMax2: function() {
      return parseInt((new Date().getTime() / 1000).toFixed(0));
    }
  },
  methods: {
    fetchData () {
      //Fetches data from MySQL server instead! :)
      //@todo Ändra från localhost till den aktuella platsen för PHP-filen på servern! 
      axios.get('http://localhost/retrieveMachineUsageInfoFromMySQL_ALL_DATA_FOR_VISUALIZATION.php', { params: {kitchen_id: this.kitchenName}})
      .then((response) => {
        if(response.data) {
          // console.log(response.data)
          this.data = response.data;
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
      console.log(ret);
      return ret;
    },
    setInfo(d) {
      this.info= `${d.machineName} ${this.getDate(d.start)} ${this.getDuration(d.duration)}` 
      this.infoLocation = {left: d.pageX, top: d.pageY-5}
    },
    getDate(time) {
      const date = new Date((time + Number.parseInt(this.offset)) * 1000)
      return `${date.getDate()}/${date.getMonth()+1} ${date.getHours()}:${date.getMinutes()}`
    },
    getDuration(time) {
      return `${Math.floor(time/3600)} h, ${Math.floor(time%3600/60)} min`
    },
    changeGrafanaURL() {
      this.grafanaSrc = this.grafanaStart;
      console.log(this.offset)
    }
  }
}
</script>
<style>
  body {
    font-family: sans-serif;
    font-size: 10pt;
    margin-top: 45px;
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

  #grafana {
    position: absolute;
    top:0;
    left: 0;
    z-index: -1;
    margin-left: 80px;
    margin-top: 100px;
    width: 97%;
  }

  #machineNamesList {
  }

  #legendContainer {
  }

  legend {
    padding-left: 6.25%;
    background: white;
    width: 97%;
  }

  input {
  }

  #slider {
  }

  foodstats {
    position: relative;
    margin-left: 100px;
  }
</style>