<template>
  <div>
    <video ref="video" autoplay></video>
    <canvas ref="canvas"></canvas>
    <button @click="toggleDrawFace">{{ isDrawingFace ? 'Stop Drawing' : 'Draw Face' }}</button>
  </div>
</template>

<script>
import * as faceapi from 'face-api.js';

export default {
  name: 'Camera',
  data() {
    return {
      isDrawingFace: false,
    };
  },
  mounted() {
    this.loadModels();
  },
  methods: {
    async loadModels() {
      const MODEL_URL = '/models'; // Ensure you have these models in the public/models folder
      await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
      await faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL);
      this.startCamera();
    },
    async startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = stream;
        this.$refs.video.addEventListener('loadeddata', this.drawToCanvas);
      } catch (err) {
        console.error("Error accessing the camera: ", err);
      }
    },
    drawToCanvas() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      const context = canvas.getContext('2d');

      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      const draw = async () => {
        context.drawImage(video, 0, 0, canvas.width, canvas.height);

        if (this.isDrawingFace) {
          const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions()).withFaceLandmarks();
          faceapi.draw.drawDetections(canvas, detections);
          faceapi.draw.drawFaceLandmarks(canvas, detections);
        }

        requestAnimationFrame(draw);
      };

      draw();
    },
    toggleDrawFace() {
      this.isDrawingFace = !this.isDrawingFace;
    }
  }
}
</script>

<style scoped>
video, canvas {
  display: block;
  width: 100%;
  max-width: 640px;
  margin: 0 auto;
}
button {
  display: block;
  margin: 20px auto;
  padding: 10px 20px;
  font-size: 16px;
}
</style>
