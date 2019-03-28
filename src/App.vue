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
          :locationIndex="locationIndex" />
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
        <!-- <footer class="sticky">
          <p>
            Hier kan tekst!
          </p>
        </footer> -->
        <div class="page"></div>
        <div class="end" ref="end"></div>
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

export default {
  name: 'app',
  components: {
    Year,
    Timeline
  },
  data () {
    return {
      yearData: undefined,
      currentYear: undefined,
      locationIndex: undefined,
      // progress: undefined,
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
    locationIndex: function () {
      // if (this.map.isMoving()) {
      // // if (this.locationIndex > 1 || this.locationIndex < this.yearData.locations.length - 1) {
      //   return
      // }

      // let bounds = new mapboxgl.LngLatBounds()

      // let fromIndex = Math.max(0, this.locationIndex - 2)
      // let toIndex = Math.min(this.yearData.locations.length, this.locationIndex + 2)

      // // if (this.locationIndex === 0 || this.locationIndex === this.yearData.locations.length - 1) {
      //   fromIndex = 0
      //   toIndex = this.yearData.locations.length
      // // }

      // if (this.locationIndex === 0) {
      //   this.map.flyTo({
      //     center: this.yearData.locations[0].geometry.coordinates,
      //     zoom: 10,
      //     duration: 2000,
      //     speed: 0.3,
      //     // linear: true,
      //     padding: 100,
      //     // maxZoom: 12
      //   })

      // } else {
      //   this.yearData.locations
      //     .slice(fromIndex, toIndex).forEach((location) => {
      //       bounds = bounds.extend(location.geometry.coordinates)
      //       location.properties.triangles.forEach((triangle) => {
      //         triangle.forEach((point) => {
      //           bounds = bounds.extend(point.coordinates)
      //         })
      //       })
      //     })

      //   this.map.fitBounds(bounds, {
      //     duration: 2000,
      //     speed: 0.3,
      //     // linear: true,
      //     padding: 100,
      //     maxZoom: 12
      //   })
      // }
    },
    currentYear: function () {
      let bounds = new mapboxgl.LngLatBounds()

      this.yearData.locations.forEach((location) => {
          bounds = bounds.extend(location.geometry.coordinates)
          location.properties.triangles.forEach((triangle) => {
            triangle.forEach((point) => {
              bounds = bounds.extend(point.coordinates)
            })
          })
        })

      this.map.fitBounds(bounds, {
        duration: 2000,
        speed: 0.3,
        // linear: true,
        padding: 100,
        maxZoom: 12
      })
    },
    triangles: function () {
      this.map.getSource('triangles').setData(this.triangles)
    },
    locations: function () {
      this.map.getSource('locations').setData(this.locations)

      const bounds = new mapboxgl.LngLatBounds()
      this.locations.features.forEach((feature) => {
        bounds.extend(feature.geometry.coordinates)
      })
      this.bounds = bounds

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
          locations
        }))
    }
  },
  methods: {
    end: function () {
      if (this.bounds) {
        this.map.fitBounds(this.bounds, {
          duration: 2000,
          speed: 0.3,
          // linear: true,
          padding: 20,
          maxZoom: 12
        })
      }
    },
    yearProgress: function (year, progress) {
      if (this.currentYear !== year) {
        this.currentYear = year
        this.yearData = this.years.filter((year) => year.year === this.currentYear)[0]
      }

      const count = this.yearData.locations.length
      const index = Math.min(Math.floor((count + 1) * progress) - 1, count - 1)

      this.locationIndex = index

      let timestamp
      if (index < 0) {
        timestamp = new Date(this.yearData.locations[0].properties.date).getTime() - 1
      } else {
        timestamp = new Date(this.yearData.locations[this.locationIndex].properties.date).getTime()
      }

      this.setTimestamp(timestamp)
    },
    setTimestamp: function (timestamp) {
      this.map.setFilter('locations', ['<=', 'timestamp', timestamp])
      this.map.setFilter('triangles', ['<=', 'timestamp', timestamp])
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
        paint: {
          'line-color': 'black',
          'line-width': 3,
          'line-opacity': .8
        }
      })

      this.map.addLayer({
        id: 'locations',
        type: 'circle',
        source: 'locations',
        paint: {
          'circle-radius': 7,
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

      const from = -7000000000000
      this.setTimestamp(from)
    }
  },
  created: function () {
    window.addEventListener('keypress', this.keyPress)
  },
  mounted: function () {
    this.scrollMagic = new ScrollMagic.Controller()

    const scene = new ScrollMagic.Scene({
      triggerElement: this.$refs.end,
      duration: 0
    })

    this.scrollMagic.addScene(scene)
    scene.on('enter', this.end)

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
  background-color: black;
  width: 33.33%;
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
