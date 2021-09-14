<template>
  <div id="foodStatsWrapper">
    Från: <input id="datePickerFrom" type="date" :value="pickedDateFrom" v-on:input="showDateFrom">
    Till: <input id="datePickerTo" type="date" :value="pickedDateTo" v-on:input="showDateTo">
    <input id="changeTimeIntervalButton" type="button" value="Ändra tidsspann" v-on:click="changeTimeInterval">
  </div>
</template>

<script>
export default {
  name: 'Foodstats',
  props: {},
  data: function () {
    return {
      pickedDateFrom: '2021-09-01',
      pickedDateTo: '2021-09-02',
    }
  },
  computed: {},
  methods: {
    showDateFrom(){
      this.pickedDateFrom = document.getElementById("datePickerFrom").value;
      var datum = Date.parse(this.pickedDateFrom)/1000;
      console.log(this.pickedDateFrom);
      console.log(datum);
    },
    showDateTo(){
      this.pickedDateTo = document.getElementById("datePickerTo").value;
      var datum = Date.parse(this.pickedDateTo)/1000;
      console.log(this.pickedDateTo);
      console.log(datum);
    },
    changeTimeInterval(){
      if(this.pickedDateTo > this.pickedDateFrom) {
        alert("valid!");
      }
      else {
        alert("Datumet Från måste vara innan Till!");
      }
    }
  }
}
</script>

<style scoped>
#foodStatsWrapper {
  position: absolute;
  margin-left: 1300px;
  margin-top: -100px;
}
</style>
