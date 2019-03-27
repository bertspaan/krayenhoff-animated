<template>
  <div id="app" :class="{
      scrolling: scrolling
    }">
    <div class="panels">
      <div class="map-container">
        <div id="map" />
      </div>
      <div class="timeline-container">
        <Timeline :years="years" :currentYear="currentYear"
          :progress="progress" />
      </div>
    </div>

    <main>
      <div class="text">
        <div class="page"></div>
        <header class="sticky">
          <h1>Krayenhoff Animated</h1>
          <p class="dutch">
            Deze animatie toont de chronologische volgorde van Kraijenhoffs metingen.
            De volgorde is gebaseerd op Kraijenhoffs eigen wetenschappelijke notitieboeken
            uit de collectie van de Universiteitsbibliotheek Leiden.</p>
          <!-- <h1>Krayenhoff Animated</h1> -->
          <p class="english">
            This animation shows Kraijenhoff's measurements in chronological order.
            The order is taken from Kraijenhoff's own scientific notebooks housed in
            the collection of Leiden University Libraries.
          </p>
        </header>
        <div class="page"></div>
      </div>

      <Year v-for="year in years" :key="year.year"
        :scrollMagic="scrollMagic" @progress="yearProgress"
        :year="year" />

      <div class="text">
        <div class="page"></div>
        <footer class="sticky">
          <p>
            Lorem ipsum dolor sit amet consectetur adipisicing elit. Veniam, nisi.
            Aspernatur debitis consequuntur error ad cupiditate laborum placeat doloribus
            necessitatibus, exercitationem iste, quod dicta nulla aut aliquid non
            esse reiciendis.
          </p>
        </footer>
        <div class="page"></div>
      </div>
    </main>
  </div>
</template>

<script>
const axios = require('axios')
const ScrollMagic = require('scrollmagic')

import Year from './components/Year.vue'
import Timeline from './components/Timeline.vue'

// function fitBounds (bounds) {
//   if (!bounds) {
//     return
//   }

//   map.fitBounds(bounds, {
//     padding: 100,
//     maxZoom: 16
//   })
// }
// function flyTo (center, zoom) {
//   map.flyTo({
//     center: center,
//     zoom: zoom
//   })
// }
//   const time = {
//     start: performance.now(),
//     total: 2000
//   }
//   window.scrollTo(0, 0)
//   function pageScroll () {
//     // time.elapsed = now - time.start
//     // const progress = time.elapsed / time.total
//     window.scrollBy(0, 2)
//     // if (progress < 1) requestAnimationFrame(tick)
//     // window.requestAnimationFrame(pageScroll)
//   }
//   pageScroll()
// })
// function initializeScrollMagic () {
//   var controller = new ScrollMagic.Controller();
//   var scene = new ScrollMagic.Scene({
//       duration: locations.features.length * 1000,
//       offset: 50
//     })
//     .addTo(controller)
//   map.setFeatureState({source: 'locations-source', id: 1}, { hond: true})
//   scene.on('progress', function (event) {
//     var show = Math.round(locations.features.length * event.progress)
//     for (var i = 0; i < show; i++) {
//       map.setFeatureState({source: 'locations-source', id: i}, { visible: true})
//     }
//     for (var i = show; i < locations.features.length; i++) {
//       map.setFeatureState({source: 'locations-source', id: i}, { visible: false})
//     }
//   })
// }

export default {
  name: 'app',
  components: {
    Year,
    Timeline
  },
  data () {
    return {
      currentYear: undefined,
      progress: undefined,
      locations: null,
      triangles: null,
      map: undefined,
      scrollMagic: undefined,
      scrolling: false
    }
  },
  watch: {
    scrolling: function () {
      if (this.scrolling === true) {
        const time = {
          start: performance.now(),
          total: 2000
        }
        window.scrollTo(0, 0)

        function pageScroll () {
          // const now = performance.now()
          // time.elapsed = now - time.start
          // const progress = time.elapsed / time.total

          window.scrollBy(0, 25)
          // if (progress < 1) {
            // window.requestAnimationFrame(pageScroll)
          // }
        }

        window.setInterval(pageScroll, 200)

        // pageScroll()
      }
    },
    currentYear: function () {
      let bounds = new mapboxgl.LngLatBounds()

      // this.years[this.currentYear].forEach((location) => {
      //   bounds = bounds.extend(location.geometry.coordinates)
      // })

      // this.map.fitBounds(bounds, {
      //   duration: 10000
      // })
    },
    triangles: function () {
      this.map.getSource('triangles').setData(this.triangles)
    },
    locations: function () {
      this.map.getSource('locations').setData(this.locations)

      const triangles = this.locations.features
        .map((feature) => feature.properties.triangles.map((triangles) => ({
          type: 'Feature',
          properties: {
            timestamp: new Date(feature.properties.date).getTime()
          },
          geometry: {
            type: 'LineString',
            coordinates: [
              feature.geometry.coordinates,
              triangles[0].coordinates,
              triangles[1].coordinates,
              feature.geometry.coordinates
            ]
          }
        })))
        .flat()

      this.triangles = {
        type: 'FeatureCollection',
        features: triangles
      }
    }
  },
  computed: {
    years: function () {
      if (!this.locations) {
        return []
      }

      function groupBy (xs, fn) {
        return xs.reduce((rv, x) => {
          (rv[fn(x)] = rv[fn(x)] || []).push(x)
          return rv
        }, {})
      }

      const grouped = groupBy(this.locations.features, (feature) => feature.properties.series)

      return Object.entries(grouped)
        .map(([year, locations]) => ({
          year: parseInt(year),
          firstTimestamp: this.firstTimestamp(locations),
          days: this.days(locations),
          locations
        }))
        .map((year) => ({
          ...year,
          totalDays: year.days[year.days.length - 1],
          positions: this.positions(year.days)
        }))
        .map((year) => ({
          ...year,
          totalPositions: year.positions[year.positions.length - 1]
        }))
    }
  },
  methods: {
    firstTimestamp: function (locations) {
      return new Date(locations[0].properties.date).getTime()
    },
    positions: function (cumulativeDays) {
      const minDays = 8
      const maxDays = 25

      let previousDays = 0
      let previousPosition = 0
      return cumulativeDays.map((days, index) => {
        let position = 0
        if (index > 0) {
          position = Math.max(minDays, Math.min(days - previousDays, maxDays))
        }

        previousDays = days

        const cumulativePosition = position + previousPosition
        previousPosition = cumulativePosition

        return cumulativePosition
      })
    },
    days: function (locations) {
      const firstDate = new Date(locations[0].properties.date)

      const cumulativeDays = locations
          .map((location) => {
            const date = new Date(location.properties.date)
            return (date - firstDate) / 1000 / 60 / 60 / 24
          })

      return cumulativeDays
    },
    yearProgress: function (year, timestamp, progress) {
      if (this.currentYear !== year) {
        this.currentYear = year
      }

      this.progress = progress
      this.setTimestamp(timestamp)
    },
    setTimestamp: function (timestamp) {
      this.map.setFilter('locations', ['<', 'timestamp', timestamp])
      this.map.setFilter('triangles', ['<', 'timestamp', timestamp])
    },
    loadLocations: function () {
       axios
        .get('krayenhoff.geojson')
        .then((response) => {
          const locations = response.data

          this.locations = {
            type: 'FeatureCollection',
            features: locations.features.map((feature) => ({
              ...feature,
              properties: {
                ...feature.properties,
                timestamp: new Date(feature.properties.date).getTime()
              }
            }))
          }
        })
    },
    keyPress: function (event) {
      if (event.key === 's') {
        window.setTimeout(() => {
          this.scrolling = true
        }, 2000)
      }

      if (event.key === 'c') {
        this.scrolling = false
      }
    },
    addLayers: function () {
      this.map.addSource('locations', {
        type: 'geojson',
        data: this.locations
      })

      this.map.addSource('triangles', {
        type: 'geojson',
        data: this.triangles
      })

      this.map.addLayer({
        id: 'triangles',
        type: 'line',
        source: 'triangles',
        layout: {
          // 'line-join': 'round',
          // 'line-cap': 'round'
        },
        paint: {
          'line-color': '#1a2b3c',
          'line-width': 2,
          'line-opacity': .8
        }
      })

      this.map.addLayer({
        id: 'locations',
        type: 'circle',
        source: 'locations',
        layout: {
          // 'line-join': 'round',
          // 'line-cap': 'round'
        },
        paint: {
          'circle-radius': 5,
          'circle-color': '#ed1c24',
          'circle-opacity': ['case',
            ['boolean', ['feature-state', 'visible'], true],
            1,
            0
          ],
          'circle-opacity-transition': {
            'duration': 3000
          }
        }
      })

      // TODO: from data!
      const from = -6364662400000
      this.setTimestamp(from)
    }
  },
  created: function () {
    window.addEventListener('keypress', this.keyPress)
  },
  mounted: function () {
    this.scrollMagic = new ScrollMagic.Controller()

    // eslint-disable-next-line
    const map = new mapboxgl.Map({
      container: 'map',
      style: {
        version: 8,
        sources: {
          'krayenhoff-map': {
            type: 'raster',
            tiles: [
              'http://mapwarper.net/mosaics/tile/1054/{z}/{x}/{y}.png'
            ],
            tileSize: 256
          }
        },
        layers: [{
          id: 'krayenhoff-map',
          type: 'raster',
          source: 'krayenhoff-map',
          minzoom: 0,
          maxzoom: 14
        }]
      },
      minZoom: 6,
      maxZoom: 12,
      maxBounds: [[1.5, 49.8], [8.5, 55.0]],
      center: [4.7779, 52.1500],
      zoom: 11
    })

    // map.scrollZoom.disable()
    // map.boxZoom.disable()
    // map.dragRotate.disable()
    // map.touchZoomRotate.disable()
    // map.dragRotate.disable()

    map.on('load', () => {
      this.addLayers()
      this.loadLocations()
    })

    this.map = map
  }
}
</script>

<style>
@import './assets/fonts.css';

html, body {
  background-color: white;
}

#app {
  width: 100%;
  height: 100vh;
}

.panels {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: space-between;
}

.map-container {
  height: 100%;
  width: 100%;
}

#map {
  height: 100%;
  width: 100%;
}

#app.scrolling {
  cursor: none;
}

::-webkit-scrollbar {
  display: none;
}

.timeline-container {
  background-color: #1a2b3c;
  width: 500px;
  flex-shrink: 0;
}

main {
  width: 100%;
  height: 100%;
  z-index: 1000;
  position: relative;
  padding: 1em;
  box-sizing: border-box;
  pointer-events: none;
}

header, footer {
  width: 60%;
  padding: 1em;
  box-sizing: border-box;
  background-color: white;
}

.text {
  position: relative;
}

.sticky {
  position: sticky;
  top: 1em;
}

.page {
  height: 100vh;
}

</style>
