<template>
  <section>
    <div ref="trigger"></div>
    <div :style="{
      height: `${scrollLength}px`
    }">
    </div>
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
      pixelsPerDay: 25,
      pixelsPerLocation: 400
    }
  },
  computed: {
    scrollLength: function () {
      return (this.year.locations.length + 2) * this.pixelsPerLocation
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
      const progress = this.padProgress(event.progress, 0)

      const daysProgress = this.year.totalDays * 24 * 60 * 60 * 1000 * progress
      const timestamp = this.year.firstTimestamp + progress

      // this.$emit('progress', this.year.year, timestamp, progress)
      this.$emit('progress', this.year.year, progress)
    }
  },
  mounted: function () {
    const scene = new ScrollMagic.Scene({
      triggerElement: this.$refs.trigger,
      duration: this.scrollLength
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
