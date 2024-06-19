<script setup>
import { ref } from 'vue';
import axios from 'axios';

const inputData = ref('');
const qrCode = ref(null);

const generateQRCode = async () => {
  try {
    const response = await axios.post('http://192.168.1.10:3001/generate', { data: inputData.value });
    qrCode.value = response.data.qrCode;
  } catch (error) {
    console.error("There was an error generating the QR code: ", error);
  }
};
import { RouterLink} from 'vue-router'

</script>

<template>
  <div>
    <input v-model="inputData" placeholder="Enter data" />
    <button @click="generateQRCode">Generate QR Code</button>
    <img v-if="qrCode" :src="qrCode" />
  </div>
  <RouterLink to="/qrcode">scanner</RouterLink>
</template>
