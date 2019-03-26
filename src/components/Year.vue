<template>
  <section>
    <div ref="trigger"></div>
    <div :style="{
      height: `${year.totalDays * pixelsPerDay}px`
    }">
    </div>
    <!--
    <ol>
      <li v-for="(location, index) in book.locations" :key="location.properties.date"
        :style="{
          height: `${book.days[index]}vh`
        }">
        {{ location.properties.place }}: {{ location.properties.date }}
      </li>
    </ol> -->
    <div class="spacer"></div>
  </section>
</template>

<script>
const ScrollMagic = require('scrollmagic')

export default {
  name: 'Year',
  props: {
    year: Object,
    scrollMagic: Object
  },
  data () {
    return {
      pixelsPerDay: 25
    }
  },
  methods: {
    padProgress: function (progress, padding=.2) {
      let padded = 0
      if (progress > padding && progress <= (1 - padding)) {
        padded = (progress - padding) / (1 - 2 * padding)
      } else if (progress > (1 - padding)) {
        padded = 1
      }

      return padded
    },
    progress: function (event) {
      const progress = this.padProgress(event.progress)

      const daysProgress = this.year.totalDays * 24 * 60 * 60 * 1000 * progress
      const timestamp = this.year.firstTimestamp + progress

      this.$emit('progress', this.year.year, timestamp, progress)
    }
  },
  mounted: function () {

    const scene = new ScrollMagic.Scene({
      triggerElement: this.$refs.trigger,
      duration: this.year.totalDays * this.pixelsPerDay
    })

    scene.on('progress', this.progress)

    this.scrollMagic.addScene(scene)
    this.scene = scene
  }
}
</script>

<style scoped>
.spacer {
  height: 20vh;
}
</style>
