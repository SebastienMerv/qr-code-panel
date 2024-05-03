<template>
  <div class="bg-gray-100 min-h-screen flex flex-row items-start justify-center p-8">
    <!-- Formulaire à gauche -->
    <div class="flex flex-col items-start space-y-4 mr-8">
      <h1 class="text-3xl font-semibold mb-4">Génération de QR CODE</h1>
      <div class="flex flex-col space-y-4">
        <label for="size" class="text-lg">Taille du QR CODE</label>
        <input type="range" id="size" @input="updateQrCode" min="0" max="100" value="80">
      </div>
      <div class="flex flex-col space-y-4">
        <label for="file" class="text-lg">Votre logo</label>
        <input type="file" id="file" @change="updateQrCode">
      </div>
      <div class="flex flex-col space-y-4">
        <label for="link" class="text-lg">Lien</label>
        <input type="text" id="link" @input="updateQrCode">
      </div>
    </div>
    <!-- QR code à droite -->
    <div class="qr w-96 flex items-center justify-center flex-col">
      <canvas id="qrCanvas" class="w-64 h-64 bg-white"></canvas>

      <a class="text-blue-500 underline" href="#" @click="download" target="_blank">Télécharger</a>
    </div>
  </div>
</template>

<script setup>
import QrCode from 'qrcode';
import { onMounted } from 'vue';

onMounted(() => {
  updateQrCode();
});

function download() {
  const qrCanvas = document.getElementById('qrCanvas');
  const link = document.createElement('a');
  link.download = 'qr-code.png';
  link.href = qrCanvas.toDataURL('image/png');
  link.click();

}

function updateQrCode() {
  const qrCanvas = document.getElementById('qrCanvas');
  const size = document.getElementById('size').value;
  const file = document.getElementById('file').files[0];
  const link = document.getElementById('link').value || 'https://bitjson.com/';

  const context = qrCanvas.getContext('2d');
  context.clearRect(0, 0, qrCanvas.width, qrCanvas.height);

  QrCode.toCanvas(qrCanvas, link, {
    errorCorrectionLevel: 'H',
    margin: 4,
    scale: size / 20,
    version: 10,
    color: { dark: '#000', light: '#fff' },
    background: '#fff',
  });

  if (file) {
    const reader = new FileReader();
    reader.onload = function (e) {
      const logoImg = new Image();
      logoImg.onload = function () {
        const logoSize = qrCanvas.width * 0.2;
        const logoX = (qrCanvas.width - logoSize) / 2;
        const logoY = (qrCanvas.height - logoSize) / 2;
        context.fillStyle = '#fff';
        context.fillRect(logoX, logoY, logoSize, logoSize);
        context.drawImage(logoImg, logoX, logoY, logoSize, logoSize);
      };
      logoImg.src = e.target.result;
    };
    reader.readAsDataURL(file);
  }
}
</script>
