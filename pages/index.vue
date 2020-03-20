<template>
  <div v-if="data" class="container">
    <ul class="countries">
      <li :class="['country', {selected: countries.length === selectedCountries.length}]" @click="selectWorld">
        Mundo
      </li><li v-for="country in countries" :key="country" :class="['country', {selected: selectedCountries.includes(country)}]" @click="selectCountry(country)">
        {{ country }}
      </li>
    </ul>
    <section>
      <div>
        <!-- Confirmados: {{ confirmed }} -->
      </div>
      <div>
        <!-- Muertes: {{ deaths }} -->
      </div>
      <div>
        <!-- Recuperados: {{ recovered }} -->
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
          <highcharts :options="statsSpeedChart" />
        </div>
      </div>

      <div class="charts">
        <div class="chart">
          <highcharts :options="compareChart" />
        </div>
      </div>

      <div class="charts">
        <div class="chart">
          <highcharts :options="relationChart" />
        </div>
      </div>
      <pre>
        {{ stats }}
        <!-- {{ dates }} -->
        <!-- {{ timestamps }} -->
      </pre>
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
      selectedCountries: [],
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
      statsSpeedChart: {
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
          text: 'Casos'
        },
        series: [{
          data: null
        }]
      },
      relationChart: {
        chart: {
          type: 'spline',
          backgroundColor: '#FAFAFA'

        },
        title: {
          text: 'Relaciones'
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
    timestamps () {
      const timestamps = this.countries
        .map(country => this.data[country].map(data => (new Date(data.date)).getTime()))
        .reduce((last, current) => [...last, ...current], [])
        .sort()
      return [...new Set(timestamps)]
    },
    dates () {
      return this.timestamps.reduce((last, current) => {
        const date = new Date(current)
        const y = date.getFullYear()
        const m = date.getMonth() + 1
        const d = date.getDate()
        last[current] = {
          date: `${y}-${m}-${d}`,
          confirmed: {
            amount: 0,
            speed: 0
          }
        }
        return last
      }, {})
    },
    stats () {
      const dates = Object.keys(this.dates).map((_date, index, arr) => {
        const date = this.dates[_date]
        const lastDate = this.dates[arr[index - 1]]

        const countries = this.countries.filter(country => this.selectedCountries.includes(country))

        const amounts = countries
          .map((country) => {
            return this.data[country]
              .find(data => data.date === date.date)
          })
          .reduce((last, current) => {
            return {
              confirmed: last.confirmed + current.confirmed,
              deaths: last.deaths + current.deaths,
              recovered: last.recovered + current.recovered
            }
          }, { confirmed: 0, deaths: 0, recovered: 0 })

        const lastAmounts = lastDate ? countries
          .map((country) => {
            return this.data[country]
              .find(data => data.date === lastDate.date)
          })
          .reduce((last, current) => {
            return {
              confirmed: last.confirmed + current.confirmed,
              deaths: last.deaths + current.deaths,
              recovered: last.recovered + current.recovered
            }
          }, { confirmed: 0, deaths: 0, recovered: 0 }) : { confirmed: 0, deaths: 0, recovered: 0 }

        const speeds = {
          confirmed: amounts.confirmed - lastAmounts.confirmed,
          deaths: amounts.deaths - lastAmounts.deaths,
          recovered: amounts.recovered - lastAmounts.recovered
        }

        return {
          date: date.date,
          confirmed: {
            amount: amounts.confirmed,
            speed: speeds.confirmed
          },
          deaths: {
            amount: amounts.deaths,
            speed: speeds.deaths
          },
          recovered: {
            amount: amounts.recovered,
            speed: speeds.recovered
          }
        }
      })
      return dates
    }

    // "1581562800000": {
    //   "date": "2020-2-13",
    //   "confirmed": {
    //     "amount": 0,
    //     "speed": 0
    //   }
    // },

    // Thailand: [
    //   { date: "2020-1-22", confirmed: 2, deaths: 0, recovered: 0 },
    //   { date: "2020-1-23", confirmed: 3, deaths: 0, recovered: 0 },
    // ],
    // Thailand: [
    //   { date: "2020-1-22", confirmed: 2, deaths: 0, recovered: 0 },
    //   { date: "2020-1-23", confirmed: 3, deaths: 0, recovered: 0 },
    // ]
    // zone () {
    //   const countries = Object.keys(this.data)
    //     .map(name => ({ name, data: this.data[name] }))
    //     .filter(country => this.selected.includes(country.name))

    //   return countries
    // },
    // zoneByDate () {
    //   return this.countriesByDate(this.zone)
    // },
    // confirmed () {
    //   return this.lastData && this.lastData.confirmed
    // },
    // deaths () {
    //   return this.lastData && this.lastData.deaths
    // },
    // recovered () {
    //   return this.lastData && this.lastData.recovered
    // },
    // lastData () {
    //   const datesList = this.stats
    //   stat lastDate = datesList[datesList.length - 1]

    //   stat.confirmed.amount]
    // }
  },
  mounted () {
    this.updateData(this.selectWorld)
    setInterval(this.updateData, 1000 * 60 * 5)
  },
  methods: {
    updateData (cb) {
      this.$axios.$get('https://pomber.github.io/covid19/timeseries.json').then((data) => {
        this.data = data
        cb && cb()
      })
    },
    selectWorld () {
      this.selectedCountries = (this.selectedCountries.length === this.countries.length ? [] : [...this.countries])
      this.selectCountry()
    },
    selectCountry (zone) {
      this.selectedCountries = this.selectedCountries.includes(zone)
        ? [...this.selectedCountries.filter(name => name !== zone)]
        : [...this.selectedCountries, zone]
          .filter(_ => _ !== undefined)

      this.confirmedChart.series = [
        {
          data: this.stats.map((stat) => {
            return stat.confirmed.amount
          }),
          name: 'Confirmados',
          color: '#0061f2'
        },
        {
          data: this.stats.map((stat) => {
            return stat.confirmed.speed
          }),
          name: 'Velocidad de Confirmados',
          color: '#f4a100'
        }
      ]

      this.deathsChart.series = [
        {
          data: this.stats.map((stat) => {
            return stat.deaths.amount
          }),
          name: 'Muertes',
          color: '#e81500'
        },
        {
          data: this.stats.map((stat) => {
            return stat.deaths.speed
          }),
          name: 'Velocidad de Muertes',
          color: '#f4a100'
        }
      ]

      this.recoveredChart.series = [
        {
          data: this.stats.map((stat) => {
            return stat.recovered.amount
          }),
          name: 'Recuperados',
          color: '#00ac69'
        },
        {
          data: this.stats.map((stat) => {
            return stat.recovered.speed
          }),
          name: 'Velocidad de Recuperados',
          color: '#f4a100'
        }
      ]

      this.compareChart.series = [
        {
          data: this.stats.map((stat) => {
            return stat.confirmed.amount
          }),
          name: 'Confirmados',
          color: '#0061f2'
        },
        {
          data: this.stats.map((stat) => {
            return stat.deaths.amount
          }),
          name: 'Muertes',
          color: '#e81500'
        },
        {
          data: this.stats.map((stat) => {
            return stat.recovered.amount
          }),
          name: 'Recuperados',
          color: '#00ac69'
        }
      ]

      this.statsSpeedChart.series = [
        {
          data: this.stats.map((stat) => {
            return stat.confirmed.speed
          }),
          name: 'Confirmados',
          color: '#0061f2'
        },
        {
          data: this.stats.map((stat) => {
            return stat.deaths.speed
          }),
          name: 'Muertes',
          color: '#e81500'
        },
        {
          data: this.stats.map((stat) => {
            return stat.recovered.speed
          }),
          name: 'Recuperados',
          color: '#00ac69'
        }
      ]

      this.relationChart.series = [
        {
          data: this.stats.map((stat) => {
            return stat.recovered.amount ? Math.round(stat.recovered.amount / stat.confirmed.amount * 1000) / 10 : 0
          }),
          name: '% Recuperación',
          color: '#00ac69'
        },
        {
          data: this.stats.map((stat) => {
            return stat.deaths.amount ? Math.round(stat.deaths.amount / stat.confirmed.amount * 1000) / 10 : 0
          }),
          name: '% Mortalidad',
          color: '#e81500'
        }
      ]
    }
    // countriesByDate (countries) {
    //   const dates = {
    //     countries: this.selected,
    //     stats: {}
    //   }
    //   countries.forEach((country) => {
    //     country.data.forEach((data, index, arr) => {
    //       const date = dates.stats[data.date]

    //       dates.stats[data.date] = {
    //         confirmed: (date ? date.confirmed : 0) + data.confirmed,
    //         deaths: (date ? date.deaths : 0) + data.deaths,
    //         recovered: (date ? date.recovered : 0) + data.recovered
    //       }
    //     })
    //   })

    //   const datesList = Object.keys(dates.stats)

    //   datesList.forEach((date, index, arr) => {
    //     dates.stats[date].confirmedSpeed = dates.stats[date].confirmed - (arr[index - 1] ? dates.stats[arr[index - 1]].confirmed : 0)
    //     dates.stats[date].deathsSpeed = dates.stats[date].deaths - (arr[index - 1] ? dates.stats[arr[index - 1]].deaths : 0)
    //     dates.stats[date].recoveredSpeed = dates.stats[date].recovered - (arr[index - 1] ? dates.stats[arr[index - 1]].recovered : 0)
    //   })

    //   return dates
    // }
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
