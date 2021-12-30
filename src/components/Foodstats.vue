<template>
  <div id="foodStatsWrapper">
    <div id=""> Datum: <input id="datePicker" type="date" v-model="pickedDate"> </div>
    <input id="changeDateButton" type="button" value="Ladda data för valt datum" v-on:click="changeDate">
  </div>
</template>

<script>
export default {
  name: 'Foodstats',
  props: {},
  data: function() {
    return {
      pickedDate: null,
      datum: null,
      // pickedDateTo: '2021-09-02',
    }
  },
  computed: {},
  methods: {
    showDate: function(){
      console.log(this.pickedDate);
      console.log("DATUM: " + this.datum);
    },
    changeDate: function(){
      this.pickedDate = document.getElementById("datePicker").value;
      this.datum = Date.parse(this.pickedDate)/1000;

      if(this.pickedDate) {
        this.$emit("change-date", this.datum);
      }
      else {
        alert("Data saknas data för datumet");
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

#changeDateButton {
  margin-left: 1%;
  margin-top: 4%;
}
</style>
