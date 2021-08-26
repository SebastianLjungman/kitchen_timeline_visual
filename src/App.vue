<template>
  <div id="app">
    <div class="info" :style="{left: infoLocation.left+'px' , top: infoLocation.top+'px'}">
      {{info}}
    </div>
      <legend>Start <input type="range" :min="offsetInit" :max="offsetMax2" v-model="offset"></legend>
      <legend>Zoom<input type="range" min="200000" max="3000000" v-model="max"></legend>
      <legend>Graph Height<input id="slider" type="range" min="590" max="2000" v-model="graphHeight"></legend>
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

export default {
  components: {
    Timeline
  },
  data () {
    return {
      data: [],
      machines: ['frying1', 'frying2', 'frima1', 'boiler1', 'boiler2', 'boiler3', 'oven1', 'oven2', 'oven3', 'oven4', 'oven5', 'oven6', 'ovenS1', 'stoveS1', 'roughDish1', 'roughDish2', 'tunnelDish1', 'serving1'],
      offsetInit: 1620270000,
      offsetMax: 1623000000,
      offset: 1620270000,
      max: 1972000,
      info: "",
      infoLocation: {left: 0, top: 0},

      //Sebastian's additions: previous grafana from value: 1620275400000, to value: 1622270100 
      graphHeight: 900,
      grafanaSrc: "https://view.stuns.i0t.se/grafana/d/UV-lugCGz/skolkok-effekt-med-nivaer?orgId=3&from=",
      grafanaSrcDomarringen: "https://view.stuns.i0t.se/grafana/d/UV-lugCGz/skolkok-effekt-med-nivaer?orgId=3&from=",
      validKitchenNames: ["tiunda", "domarringen", "stenhagen"],
      kitchenName: null
    }
  },
  created () {
    let uri = window.location.search.substring(1); 
    let params = new URLSearchParams(uri);
    this.kitchenName = params.get("kitchen_name");
    console.log(this.kitchenName);


    this.fetchData();
  },
    mounted() {
    //Adress måste innehålla giltigt värde på kitchen_name (se validKitchenNames), t.ex.
    //http://localhost/visualization_new_school/index.html?kitchen_name=domarringen

    if(this.validKitchenNames.includes(this.kitchenName)) {
      // alert("Köket existerar!");
      // this.allMachines = this.tiundaMachines;

    }
    else {
      document.getElementById("app").style.display = "none";
      alert("Köket finns inte med i listan! Kontrollera att rätt adress har angivits!");
    }
  },
  computed: {
    grafanaStart: function(){
      if(this.kitchenName === "domarringen") {
        //Kan viewPanel=10 (domarringen) och viewPanel=9 (tiunda) användas i inbäddad länk? Nuvarande view-länk laddar hela, ej snyggt
      return this.grafanaSrcDomarringen + (this.offset * 1000) + "&to=" + (this.offset * 1000 + 2000100000) + "&theme=light&viewPanel=10"
      }
      else {
        return this.grafanaSrc + (this.offset * 1000) + "&to=" + (this.offset * 1000 + 2000100000) + "&theme=light&viewPanel=9"
      }
    },
    //Bör funka, även om nya värden inte verkar dyka upp korrekt (FELSÖK!)
    offsetMax2: function() {
      return parseInt((new Date().getTime() / 1000).toFixed(0));
    }
  },
  methods: {
    fetchData () {
      // fetch("http://user.it.uu.se/~mikla253/sebastian/storedata.php")
      // .then((response) => {
      //   if (response.ok) {
      //     return response.json();
      //   }
      //   throw response;
      // })
      // .then((json) => {
      //   this.data = json.data;
      //   console.log(json.data);
      // })
      // .catch((err) => {
      //   if (err.message) {
      //     this.setError(err.message);
      //   }
      // });

      //Fetches data from MySQL server instead! :) 
      axios.get('http://localhost/retrieveMachineUsageInfoFromMySQL_ALL DATA FOR VISUALIZATION.php', { params: {kitchen_id: this.kitchenName}})
      .then((response) => {
        if(response.data) {
          // console.log(response.data)
          this.data = response.data;
        }
      });


      // fetch("http://localhost/retrieveMachineUsageInfoFromMySQL.php")
      // .then((response) => {
      //   if (response.ok) {
      //     return response.json();
      //   }
      //   throw response;
      // })
      // .then((json) => {
      // // .then((json) => {
      // //   this.data = json.data;
      // // })
      // // .catch((err) => {
      // //   if (err.message) {
      // //     this.setError(err.message);
      // //     console.log("Error LOL");
      // //   }
      // // });
      // // let retrievedMachineUsageData = response.data;
      // // let firstClick = retrievedMachineUsageData[0];
      // console.log(json.data);
      // // this.data = [];
      //   // }
      // })
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
    width: 97%;
    /* height: 100%; */
    /* display: grid; */
    /* grid-template-columns: 80px auto; */
    /* grid-template-rows: repeat(auto-fill, 20px); */
    /* overflow: hidden; */
  }

  #machineNamesList {
    /* background: pink; */

  }
  legend {
    padding-left: 6.25%;
    background: white;
    width: 97%;
  }

  input {
    /* display: inline;
    width: auto; */
  }

  #slider {

  }
</style>