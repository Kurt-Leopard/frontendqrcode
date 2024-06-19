<template>
     <div>
        <h1 v-if="found" style="color: red;">Verify User</h1>
    </div>
    <div>
      <video id="qr-video" ref="qrVideo" @canplay="enableScanner" autoplay></video>
      <button @click="stopScanner">Stop Scanner</button>
    </div>
   
  </template>
  
  <script setup>
  import { onMounted, onUnmounted, ref } from 'vue';
  import axios from 'axios';
  import jsQR from 'jsqr';
  
  const qrVideo = ref(null);
  let videoStream = null;
  let scanInterval = null;
  const found = ref('');
  const verify= ref('');
  
  const enableScanner = () => {
    scanInterval = setInterval(scanQR, 100); // Scan every 100ms
  };
  
  const scanQR = () => {
    if (qrVideo.value && qrVideo.value.readyState === qrVideo.value.HAVE_ENOUGH_DATA) {
      const video = qrVideo.value;
      const canvasElement = document.createElement('canvas');
      canvasElement.width = video.videoWidth;
      canvasElement.height = video.videoHeight;
      const canvas = canvasElement.getContext('2d');
      
      canvas.drawImage(video, 0, 0, canvasElement.width, canvasElement.height);
      const imageData = canvas.getImageData(0, 0, canvasElement.width, canvasElement.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height, {
        inversionAttempts: "dontInvert",
      });
  
      if (code) {
        console.log('Found QR code', code.data);
        validateQRCode(code.data);
      }
    }
  };
  
  const stopScanner = () => {
    if (videoStream) {
      videoStream.getTracks().forEach(track => track.stop());
    }
    if (scanInterval) {
      clearInterval(scanInterval);
    }
  };
  
  const validateQRCode = async (qrCodeData) => {

    try {
      const response = await axios.post('http://192.168.1.10:3001/validate', { qrCodeData });
      console.log(response.data); // Handle the response from the server
      found.value=response.data.isValid;
    } catch (error) {
      console.error("Error validating QR code:", error);
    }
  };
  
  onMounted(() => {
    navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } })
      .then(function(stream) {
        videoStream = stream;
        qrVideo.value.srcObject = stream;
      }).catch(err => console.error(err));
  });
  
  onUnmounted(() => {
    stopScanner();
  });
  </script>
  