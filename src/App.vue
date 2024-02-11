<template>
  <div><Line v-if="loaded" :data="chartdata" :options="options" ref="line"/></div>
  
  <button v-on:click="updateData">Update chart</button>
  <br />
  <br />
  <div>
    <select v-model="selectedApartmentId" @change="updateData($event)">
        <option v-for="apartment in apartments" v-bind:key="apartment">{{apartment.id}}</option>
    </select>
    <span> Индекс квартиры: {{selectedApartmentId}}</span>
  </div>
  <div class="qwe">
    <Datepicker v-model="dateStart"></Datepicker>
  </div>
  <div>
    <Datepicker v-model="dateFinish"></Datepicker>
  </div>
</template>
<script>
import '@vuepic/vue-datepicker/dist/main.css';
import { Line } from 'vue-chartjs'
import Datepicker from '@vuepic/vue-datepicker';
import {
  Chart,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

Chart.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)

export default {
  name: 'App',
  components: {
    Line, Datepicker 
  },
  async mounted() {
    this.loaded = false;
    const response = await fetch("http://localhost:5000/api/Apartments/GetApartments");
    const result = await response.json();
    this.apartments = result.map((r) => { return { id: r.id, url: r.url };
});

  }, 
  data: function() {
    return {
      options: 
      {
        responsive: true,
        maintainAspectRatio: false
      },
      chartdata: null,
      loaded: false,
      apartments: [],
      selectedApartmentId: null,
      monthNames: ["January", "February", "March", "April", "May", "June",
        "July", "August", "September", "October", "November", "December"
      ],
      dateStart: new Date(2024, 0, 1),
      dateFinish: new Date(2025, 0, 1)
    }
  },
  methods: {
    updateData: async function() {
      if (this.selectedApartmentId !== null)
      {
        const response = await fetch(`http://localhost:5000/api/Apartments/GetMonthsAverageApartmentPrice?startDate=${this.dateStart.toLocaleDateString('en-US')}&finishDate=${this.dateFinish.toLocaleDateString('en-US')}&apartmentId=${this.selectedApartmentId}`);
        const result = await response.json();

        console.log(result.map((r) => { return this.monthNames[new Date(r.date).getMonth()] }));
        console.log(result.map((r) => { return r.price }));

        this.chartdata = {
          labels: result.map((r) => { return this.monthNames[new Date(r.date).getMonth()] }),
          datasets: [
            {
              label: 'Data One',
              backgroundColor: '#f87979',
              data: result.map((r) => { return r.price })
            }
          ]
        };

        this.loaded = true;
      }

      return null;
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>