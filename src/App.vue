<template>
  <div id="app">
    <!-- Janus Videos List -->
    <div v-if="!loading">
      <JanusVideo :janus="janus" :cameras="['1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15']" />
    </div>
  </div>
</template>

<script>
import Janus from './janus'

export default {
  components: {
    JanusVideo: () => import('@/components/JanusVideo')
  },
  data () {
    return {
      janus: null,
      loading: false
    }
  },
  mounted () {
    this.initJanus()
  },
  beforeDestroy () {
    this.janus.destroy()
  },
  methods: {
    initJanus () {
      this.loading = true
      let server = 'http://localhost:8088/janus'
      console.log('calling Janus init')
      Janus.init({
        // debug: 'all',
        callback: () => {
          this.janus = new Janus(
            {
              server,
              success: () => {
                this.loading = false
              },
              error: (cause) => {
                console.log(cause)
              },
              destroyed: () => {
                console.log('janus init destroyed')
              }
            })
        }
      })
    }
  }
}
</script>

<style lang="sass">
#app
  height: 90vh
  display: flex
  flex-wrap: wrap
  align-items: center
  justify-content: center
</style>
