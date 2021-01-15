<template>
  <div class="toolkit">
    <button v-if="!isRecording" @click="startRecordCanvas">record</button>
    <button v-else @click="stopRecordCanvas">stop</button>
  </div>
  <canvas ref="canvas"></canvas>
  <video ref="video" v-if="videoSrc" :src="videoSrc" controls></video>
</template>

<script>
import { onMounted, ref } from 'vue'

export default {
  setup () {
    const canvas = ref(null)
    const video = ref(null)
    const videoSrc = ref('')
    const isRecording = ref(false)
    let osc = null
    let audioStream = null
    let mediaStream = null
    let recorder = null
    let chunks = []
    const startRecordCanvas = () => {
      osc.start(0)
      mediaStream = canvas.value.captureStream(60)
      mediaStream.addTrack(audioStream.getTracks()[0])
      recorder = new (window.MediaRecorder)(mediaStream)
      recorder.ondataavailable = (e) => {
        chunks.push(e.data)
      }
      recorder.onstop = () => {
        let blob = new Blob(chunks, { type: 'video/webm' })
        chunks = []
        videoSrc.value = URL.createObjectURL(blob)
      }
      recorder.start()
      isRecording.value = true
    }

    const stopRecordCanvas = () => {
      recorder.stop()
      osc.stop(0)
      isRecording.value = false
    }

    onMounted(() => {
      let ac = new AudioContext()
      osc = ac.createOscillator()
      let dest = ac.createMediaStreamDestination()
      osc.connect(dest)
      audioStream = dest.stream
    })
    onMounted(() => {
      let cvs = canvas.value
      cvs.width = 320
      cvs.height = 240
      let ctx = cvs.getContext('2d')
      let rotate = 0
      let drawStep = () => {
        ctx.fillStyle = '#ffffff'
        ctx.fillRect(0, 0, cvs.width, cvs.height)
        ctx.translate(cvs.width / 2, cvs.height / 2)
        ctx.rotate(rotate * Math.PI / 180)
        ctx.fillStyle = '#000000'
        ctx.fillRect(-25, -25, 50, 50)
        ctx.setTransform(1, 0, 0, 1, 0, 0)
        rotate++
        window.requestAnimationFrame(() => {
          drawStep()
        })
      }
      drawStep()
    })
    return {
      canvas,
      video,
      videoSrc,
      isRecording,
      startRecordCanvas,
      stopRecordCanvas
    }
  }
}
</script>