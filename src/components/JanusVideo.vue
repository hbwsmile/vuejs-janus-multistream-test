<template>
  <!-- Janus Video -->
  <div>
    <video
      :key="camera"
      v-for="(camera, i) in cameras"
      :id="`janusVideo${i}`"
      class="janus-video"
      playsinline
      autoplay
      muted
    ></video>
  </div>
</template>

<script>
import Janus from '../janus'

export default {
  name: 'JanusVideo',
  props: {
    cameras: {
      type: Array,
      required: true
    },
    janus: {
      type: Object
    }
  },
  data () {
    return {
      streaming: []
    }
  },
  mounted () {
    this.initJanus()
  },
  methods: {
    // Init Janus
    initJanus () {
      const vm = this
      for (let i = 0; i < this.cameras.length; i++) {
        console.log(`this is iteration ${i} of the for loop`)
        this.janus.attach(
          {
            opaqueId: 'test-' + i,

            plugin: 'janus.plugin.streaming',
            success: function (pluginHandle) {
              console.log(`iteration ${i} janus attach - onSuccess called, plugin handle::`, pluginHandle)

              if (pluginHandle) {
                console.log('pluginHandle=', pluginHandle)

                vm.streaming.push({ id: i, plugin: pluginHandle })

                let body = { 'request': 'watch', id: parseInt('1') }

                console.log(`iteration ${i} sending watch request::`)

                pluginHandle.send({ 'message': body })
              }
            },
            error: function (error) { console.log(error) },
            onmessage: function (msg, jsep) {
              console.log(`iteration ${i} new message received! msg::`, msg)
              console.log(`iteration ${i} the accompanying jsep was::`, jsep)

              if (jsep !== undefined && jsep !== null) {
                console.log(`iteration ${i} jsep was not null or undefined THIS IS GOOD`)

                const foundStream = vm.streaming.find(s => s.id === i)

                if (jsep.type === 'offer') {
                  console.log(`iteration ${i} the jsep type was an offer, lets make an answer`)
                  foundStream.plugin.createAnswer(
                    {
                      jsep,
                      media: { audioSend: false, videoSend: false },
                      success: function (jsep) {
                        console.log(`iteration ${i} sending a message to request the stream starts`)
                        const body = { 'request': 'start' }

                        foundStream.plugin.send({ 'message': body, 'jsep': jsep })
                      },
                      error: function (error) {
                        Janus.error('WebRTC error:', error)
                      }
                    }
                  )
                }
              }
            },
            onremotestream: function (stream) {
              console.log(`iteration ${i} on remote stream being called`)
              const element = document.getElementById(`janusVideo${i}`)
              Janus.attachMediaStream(element, stream)
            }
          })
      }
    }
  }
}
</script>

<style scoped lang="sass">
.janus-video
  width: 350px
  height: 200px
  background: black
  border: 5px solid rgba(35, 177, 104, 0.83)
  margin: 30px 10px
</style>
