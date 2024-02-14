<template>

  <div>
    <p>Фильтр для графика</p>
    <div>Количество комнат <input v-model="roomsCountChart" placeholder="Количество комнат"/></div><br/>
  </div>

  <div><Line v-if="loaded" :data="chartdata" :options="options" ref="line"/></div>
  
  <button v-on:click="updateData">Update chart</button>
  <br />
  <br />

  <div>
    <p>Фильтр для списка</p>
    <input v-model="roomsCount" placeholder="Количество комнат"/><br/>
    <div>Показать актуальную цену: <input type="checkbox" v-model="withCurrentPrice" placeholder="Показать актуальную цену"/></div><br/>
  </div>

  <div>
    <div v-for="item in apartments" v-bind:key="item">
      <b>Номер квартиры: {{ item.id }}</b><br/>
      <b>Количество комнат: {{ item.roomsCount }}</b><br/>
      <b>Актуальная цена: {{ item.currentPrice }}</b><br/>
      <b>Дата обновления цены: {{ item.priceUpdated }}</b><br/>
      <br/>
    </div>
  </div>

  <button v-on:click="updateList">Update list</button>

</template>
<script>
import '@vuepic/vue-datepicker/dist/main.css';
import { Line } from 'vue-chartjs'
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
    Line
  },
  async mounted() {
    this.loaded = false;
    const response = await fetch("http://localhost:5000/api/Apartments/GetApartments");
    const result = await response.json();
    this.apartments = result.map((r) => { return { id: r.id, priceUpdated: r.priceUpdated, currentPrice: r.currentPrice, roomsCount: r.roomsCount };
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
      dateFinish: new Date(2025, 0, 1),
      roomsCount: 0,
      roomsCountChart: 0,
      withCurrentPrice: false
    }
  },
  methods: {
    updateList: async function() {
      const response = await fetch(`http://localhost:5000/api/Apartments/GetApartments?roomsCount=${this.roomsCount}&withCurrentPrice=${this.withCurrentPrice}`);
      const result = await response.json();
      this.apartments = result.map((r) => { return { id: r.id, priceUpdated: r.priceUpdated, currentPrice: r.currentPrice, roomsCount: r.roomsCount}});
    },

    updateData: async function() {
      let result = {};

      if (this.roomsCount !== null)
      {
        const response = await fetch(`http://localhost:5000/api/Apartments/GetMonthsAverageApartmentPrice?roomsCount=${this.roomsCountChart}`);
        result = await response.json();
      }
      else
      {
        const response = await fetch(`http://localhost:5000/api/Apartments/GetMonthsAverageApartmentPrice`);
        result = await response.json();
      }
      

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