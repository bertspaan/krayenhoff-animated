<template>
  <aside class="years">
    <ol>
      <li v-for="year in years" :key="year.year"
      :class="{
        active: currentYear === year.year
      }"
      :style="{
        height: currentYear === undefined ? `${1 / years.length * 80}%` : (currentYear === year.year ? '70%' : `45px`)
      }">
        <div class="year">
          <div class="timeline" :class="{
            active: currentYear === year.year
          }">
            <svg viewBox="0 -30 400 1000" >
              <g>
                <text v-for="(location, index) in year.locations" :key="index "
                  x="400" :y="1000 / (year.locations.length / index) + 11"
                  :class="{
                    active: index === locationIndex
                  }">
                  {{ location.properties.place }}
                </text>
              </g>
            </svg>
          </div>
          <div class="middle">
            <div class="line">
            </div>
            <div class="circle-container">
              <div v-if="currentYear === year.year && locationIndex >= 0" class="circle" :style="{
                top: `calc(${locationIndex / year.locations.length * 100}%)`
              }"></div>
            </div>
          </div>
          <div class="title">
            <h3>{{ year.year }}</h3>
            <div v-if="currentYear === year.year" class="timeline"
              :class="{
                active: currentYear === year.year
              }">
              <svg viewBox="0 -30 400 1000" >
                <g>
                  <text v-for="(location, index) in year.locations" :key="index "
                    x="0" :y="1000 / (year.locations.length / index) + 11"
                    :class="{
                      active: index === locationIndex
                    }">
                      {{ formatDate(location.properties.date, 'dutch') }}
                  </text>
                </g>
              </svg>
            </div>
          </div>
        </div>
      </li>
    </ol>
  </aside>
</template>

<script>
export default {
  name: 'Timeline',
  props: {
    years: Array,
    currentYear: Number,
    locationIndex: Number
  },
  data: function () {
    return {
      months: {
        dutch: [
          'januari',
          'februari',
          'maart',
          'april',
          'mei',
          'juni',
          'juli',
          'augustus',
          'september',
          'oktober',
          'november',
          'december'
        ]
      }
    }
  },
  methods: {
    formatDate: function (date) {
      const [month, day] = date.split('-').slice(1, 3).map((num) => parseInt(num))
      return `${day} ${this.months.dutch[month - 1]}`
    }
  }
}
</script>

<style scoped>
.years {
  padding: 1em 0;
  box-sizing: border-box;
  height: 100%;
  color: white;
}

.years ol {
  height: 100%;
  display: flex;
  justify-content: space-evenly;
  flex-direction: column;
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.years ol li {
  flex-basis: 0;
  width: 100%;
  transition: height 1s;
  box-sizing: border-box;
  padding: 5px;
}

.years ol li.active {
  flex-shrink: 0;
}

.year {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
}

.timeline {
  width: 66%;;
}

.title {
  width: 33%;
}

.timeline, .title {
  flex-shrink: 0;
}

.title {
  position: relative;
}

.title h3 {
  margin: 0 20px;
}

.middle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 14px;
  flex-shrink: 0;
  text-align: center;
  position: relative;
}

.line {
  border-left-style: solid;
  border-left-width: 4px;
  height: 100%;
}

.year.active .line {
  border-left-width: 6px;
}

.circle-container {
  position: absolute;
  top: 0;
  height: 100%;
  box-sizing: border-box;
  padding-top: 40px;
  padding-bottom: 10px;
}

.circle {
  position: relative;
  border-width: 5px;
  border-color: black;
  border-style: solid;
  height: 14px;
  width: 14px;
  background-color: #ed1c24;
  border-radius: 50%;
  transition: top .2s linear;
  transform: translate(0, 6px);
}

.timeline {
  padding-top: 40px;
  padding-bottom: 10px;

  padding-left: 20px;
  padding-right: 20px;

  text-align: right;
  opacity: 0;
  transition: opacity 0.5s;
  box-sizing: border-box;
}

.timeline.active {
  opacity: 1;
}

.timeline svg {
  height: 100%;
}

.timeline svg text {
  text-anchor: end;
  font-size: 40px;
  opacity: 0.2;
  transition: opacity 0.2s;
  fill: white;
}

.title .timeline {
  text-align: left;
  position: absolute;
  top: 0;
  height: 100%;
}

.title .timeline svg text {
  text-anchor: start;
}

.timeline svg line {
  stroke: white;
}

.timeline svg text.active {
  opacity: 1;
}
</style>