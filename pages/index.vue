<template>
  <div v-if="data" class="container">
    <ul class="countries">
      <li :class="['country', {selected: countries.length === selected.length}]" @click="selectWorld">
        Mundo
      </li><li v-for="country in countries" :key="country" :class="['country', {selected: selected.includes(country)}]" @click="selectCountry(country)">
        {{ country }}
      </li>
    </ul>
    <section>
      <div>
        Zona: {{ selected.length === countries.length ? 'Mundo' : selected.join(', ') }}
      </div>
      <div>
        Confirmados: {{ confirmed }}
      </div>
      <div>
        Muertes: {{ deaths }}
      </div>
      <div>
        Recuperados: {{ recovered }}
      </div>
      <div class="charts">
        <div class="chart">
          <highcharts :options="confirmedChart" />
        </div>
        <div class="chart">
          <highcharts :options="deathsChart" />
        </div>
        <div class="chart">
          <highcharts :options="recoveredChart" />
        </div>
      </div>

      <div class="charts">
        <div class="chart">
          <highcharts :options="statsVelocityChart" />
        </div>
      </div>

      <div class="charts">
        <div class="chart">
          <highcharts :options="compareChart" />
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { Chart } from 'highcharts-vue'

export default {
  components: {
    highcharts: Chart
  },
  data () {
    return {
      data: null,
      selected: [],
      confirmedChart: {
        chart: {
          type: 'spline',
          backgroundColor: '#FAFAFA'
        },
        title: {
          text: 'Confirmados'
        },
        series: [{
          data: null
        }]

      },
      deathsChart: {
        chart: {
          type: 'spline',
          backgroundColor: '#FAFAFA'

        },
        title: {
          text: 'Muertes'
        },
        series: [{
          data: null
        }]
      },
      recoveredChart: {
        chart: {
          type: 'spline',
          backgroundColor: '#FAFAFA'

        },
        title: {
          text: 'Recuperados'
        },
        series: [{
          data: null
        }]
      },
      statsVelocityChart: {
        chart: {
          type: 'spline',
          backgroundColor: '#FAFAFA'

        },
        title: {
          text: 'Velocidades'
        },
        series: [{
          data: null
        }]
      },
      compareChart: {
        chart: {
          type: 'spline',
          backgroundColor: '#FAFAFA'

        },
        title: {
          text: 'Velocidades'
        },
        series: [{
          data: null
        }]
      }
    }
  },
  computed: {
    countries () {
      return Object.keys(this.data).sort()
    },
    zone () {
      const countries = Object.keys(this.data)
        .map(name => ({ name, data: this.data[name] }))
        .filter(country => this.selected.includes(country.name))

      return countries
    },
    zoneByDate () {
      return this.countriesByDate(this.zone)
    },
    confirmed () {
      return this.lastData && this.lastData.confirmed
    },
    deaths () {
      return this.lastData && this.lastData.deaths
    },
    recovered () {
      return this.lastData && this.lastData.recovered
    },
    lastData () {
      const datesList = Object.keys(this.zoneByDate.stats)
      const lastDate = datesList[datesList.length - 1]

      return this.zoneByDate.stats[lastDate]
    }
  },
  mounted () {
    this.$axios.$get('https://pomber.github.io/covid19/timeseries.json').then((data) => {
      this.data = data
      this.selectWorld()
    })
  },
  methods: {
    selectWorld () {
      this.selected = (this.selected.length === this.countries.length ? [] : [...this.countries])
      this.selectCountry()
    },
    selectCountry (zone) {
      this.selected = this.selected.includes(zone)
        ? [...this.selected.filter(name => name !== zone)]
        : [...this.selected, zone]
          .filter(_ => _ !== undefined)

      this.confirmedChart.series = [
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].confirmed
          }),
          name: 'Confirmados'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].confirmedSpeed
          }),
          name: 'Velocidad de Confirmados'
        }
      ]

      this.deathsChart.series = [
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].deaths
          }),
          name: 'Muertes'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].deathsSpeed
          }),
          name: 'Velocidad de Muertes'
        }
      ]

      this.recoveredChart.series = [
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].recovered
          }),
          name: 'Recuperados'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].recoveredSpeed
          }),
          name: 'Velocidad de Recuperados'
        }
      ]

      this.compareChart.series = [
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].confirmed
          }),
          name: 'Confirmados'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].deaths
          }),
          name: 'Muertes'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].recovered
          }),
          name: 'Recuperados'
        }
      ]

      this.statsVelocityChart.series = [
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].confirmedSpeed
          }),
          name: 'Confirmados'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].deathsSpeed
          }),
          name: 'Muertes'
        },
        {
          data: Object.keys(this.zoneByDate.stats).map((date) => {
            return this.zoneByDate.stats[date].recoveredSpeed
          }),
          name: 'Recuperados'
        }
      ]
    },
    countriesByDate (countries) {
      const dates = {
        countries: this.selected,
        stats: {}
      }
      countries.forEach((country) => {
        country.data.forEach((data, index, arr) => {
          const date = dates.stats[data.date]

          dates.stats[data.date] = {
            confirmed: (date ? date.confirmed : 0) + data.confirmed,
            deaths: (date ? date.deaths : 0) + data.deaths,
            recovered: (date ? date.recovered : 0) + data.recovered
          }
        })
      })

      const datesList = Object.keys(dates.stats)

      datesList.forEach((date, index, arr) => {
        dates.stats[date].confirmedSpeed = dates.stats[date].confirmed - (arr[index - 1] ? dates.stats[arr[index - 1]].confirmed : 0)
        dates.stats[date].deathsSpeed = dates.stats[date].deaths - (arr[index - 1] ? dates.stats[arr[index - 1]].deaths : 0)
        dates.stats[date].recoveredSpeed = dates.stats[date].recovered - (arr[index - 1] ? dates.stats[arr[index - 1]].recovered : 0)
      })

      return dates
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  box-sizing: border-box;
  .countries {
    .country {
      display: inline-block;
      list-style: none;
      padding: 2px 10px;
      font-size: 10px;
      line-height: 12px;
      cursor: pointer;
      border: 1px solid transparent;
      margin: 4px;

      &.selected {
        background: lightgray;
      }

      &:hover {
        border-color: blue;
      }
    }
  }
  .charts {
    display: flex;
    .chart {
      padding: 10px;
      width: 100%;
    }
  }

  @media (max-width: 1024px) {
    .charts {
      display: block;
      .chart {
        width: 100%;
      }
    }
  }
}
</style>
