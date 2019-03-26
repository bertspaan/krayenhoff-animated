<template>
  <aside class="years">
    <ol>
      <li v-for="year in years" :key="year.year"
      :class="{
        active: currentYear === year.year
      }"
      :style="{
        // TODO: nee niet em en niet 50%! maar misschien afhankelijk van de lengte toch een beetje!
        height: currentYear === year.year ? '70%' : `2em`
      }">
        <div class="year">
          <div class="timeline" :class="{
            active: currentYear === year.year
          }">
            <svg viewBox="0 -45 400 1090" >
              <g>
                <line v-for="(position, index) in year.positions" :key="index"
                  x1="380" :y1="1000 / (year.totalPositions / position)"
                  x2="400" :y2="1000 / (year.totalPositions / position)"
                  stroke-width="7" />
              </g>
              <g>
                <text v-for="(location, index) in year.locations" :key="index "
                  x="370" :y="1000 / (year.totalPositions / year.positions[index]) + 11"
                  :class="{
                    active: index === currentIndex
                  }">
                  {{ location.properties.place }}
                </text>
              </g>
            </svg>
          </div>
          <div class="line"></div>
          <div class="title">
            <h3>{{ year.year }}</h3>
            <div v-if="currentYear === year.year">
              <div class="dutch">{{ formatDate(year.locations[currentIndex].properties.date, 'dutch') }}</div>
              <!-- <div class="english">{{ formatDate(year.locations[currentIndex].properties.date, 'english') }}</div> -->
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
    progress: Number
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
        ],
        english: [
          'January',
          'February',
          'March',
          'April',
          'May',
          'June',
          'July',
          'August',
          'September',
          'October',
          'November',
          'December'
        ]
      }
    }
  },
  methods: {
    formatDate: function (date, language) {
      const [month, day] = date.split('-').slice(1, 3).map((num) => parseInt(num))

      if (language === 'dutch') {
        return `${day} ${this.months.dutch[month - 1]}`
      }

      return `${this.months.english[month - 1]} ${day}`
    }
  },
  computed: {
    positions: function () {
      if (!this.currentYear) {
        return []
      }

      return this.years.filter((year) => year.year === this.currentYear)[0].positions
    },
    currentIndex: function () {
      const maxPosition = this.positions[this.positions.length - 1]
      const position = this.progress * maxPosition

      let index = 0
      while (this.positions[index] < position) {
        index += 1
      }

      return index
    }
  }
}
</script>

<style scoped>
.years {
  padding: 1em 0;
  box-sizing: border-box;
  height: 100%;
  pointer-events: none;
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
  justify-content: flex-end;
}

.title {
  width: 170px;
  flex-shrink: 0;
  padding: 3px 10px;
}

.title h3 {
  margin: 0;
}

.line {
  border-left-style: solid;
  border-left-width: 4px;
  height: 100%;
}

.year.active .line {
  border-left-width: 6px;
}

.timeline {
  text-align: right;
  width: 100%;
  opacity: 0;
  transition: opacity 0.5s;
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

.timeline svg text.active {
  opacity: 1;}

</style>