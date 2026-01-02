<template>
  <div class="captcha-engine">
    <div class="canvas-wrapper">
      <canvas 
        ref="captchaCanvas" 
        :width="width" 
        :height="height" 
        @click="generate"
        class="captcha-canvas"
      ></canvas>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, defineExpose } from 'vue';
import '../assets/captcha.css';

const props = defineProps({
  width: { type: Number, default: 200 },
  height: { type: Number, default: 55 }
});

const captchaCanvas = ref(null);
const captchaAnswer = ref('');
const fonts = ['bold 28px serif', 'bold 30px sans-serif', 'italic bold 26px monospace', 'bold 32px cursive'];

const getRandom = (min, max) => Math.random() * (max - min) + min;

const generate = () => {
  const canvas = captchaCanvas.value;
  const ctx = canvas.getContext('2d');
  
  // 1. Arka Plan: Rastgele Gradyan ve Doku
  const gradient = ctx.createLinearGradient(0, 0, props.width, props.height);
  gradient.addColorStop(0, `rgb(${getRandom(220, 255)}, ${getRandom(220, 255)}, ${getRandom(220, 255)})`);
  gradient.addColorStop(1, `rgb(${getRandom(180, 210)}, ${getRandom(180, 210)}, ${getRandom(180, 210)})`);
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, props.width, props.height);

  // 2. Hayalet Karakterler (Botları şaşırtmak için arka plan gürültüsü)
  ctx.globalAlpha = 0.15;
  for (let i = 0; i < 8; i++) {
    ctx.font = '24px Arial';
    ctx.fillStyle = '#000';
    ctx.fillText('ABCDEFGHJKLMNOPRSTUVYZ123456789'.charAt(getRandom(0, 31)), getRandom(0, props.width), getRandom(0, props.height));
  }
  ctx.globalAlpha = 1.0;

  let displayBody = '';
  let result = '';
  const isMath = Math.random() > 0.4; 

  if (isMath) {
    const a = Math.floor(getRandom(5, 25));
    const b = Math.floor(getRandom(1, 15));
    const isPlus = Math.random() > 0.5;
    displayBody = `${a}${isPlus ? '+' : '-'}${b}=?`;
    result = isPlus ? (a + b).toString() : (a - b).toString();
  } else {
    const pool = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
    for (let i = 0; i < 6; i++) {
      displayBody += pool.charAt(Math.floor(Math.random() * pool.length));
    }
    result = displayBody;
  }

  for (let i = 0; i < 5; i++) {
    ctx.strokeStyle = `rgba(${getRandom(50, 200)}, ${getRandom(50, 200)}, ${getRandom(50, 200)}, 0.5)`;
    ctx.lineWidth = getRandom(1, 3);
    ctx.beginPath();
    ctx.moveTo(getRandom(0, 20), getRandom(0, props.height));
    ctx.bezierCurveTo(getRandom(50, 100), getRandom(0, props.height), getRandom(100, 150), getRandom(0, props.height), props.width, getRandom(0, props.height));
    ctx.stroke();
  }

  const charArray = displayBody.split('');
  const space = props.width / (charArray.length + 1);

  charArray.forEach((char, i) => {
    ctx.font = fonts[Math.floor(Math.random() * fonts.length)];
    ctx.fillStyle = `rgb(${getRandom(0, 70)}, ${getRandom(0, 70)}, ${getRandom(0, 70)})`;
    
    const x = space * (i + 1);
    const y = props.height / 2 + getRandom(-4, 4);

    ctx.save();
    ctx.translate(x, y);
    ctx.rotate((getRandom(-25, 25) * Math.PI) / 180);
    const scale = getRandom(0.9, 1.2);
    ctx.scale(scale, scale);
    
    ctx.fillText(char, -10, 0);
    
    ctx.strokeStyle = ctx.fillStyle;
    ctx.lineWidth = 0.8;
    ctx.beginPath();
    ctx.moveTo(-15, getRandom(-8, 8));
    ctx.lineTo(15, getRandom(-8, 8));
    ctx.stroke();
    
    ctx.restore();
  });

  for (let i = 0; i < 100; i++) {
    ctx.fillStyle = `rgba(${getRandom(0, 150)}, ${getRandom(0, 150)}, ${getRandom(0, 150)}, 0.4)`;
    ctx.beginPath();
    if (Math.random() > 0.5) {
      ctx.arc(getRandom(0, props.width), getRandom(0, props.height), getRandom(0.5, 1.5), 0, Math.PI * 2);
    } else {
      ctx.rect(getRandom(0, props.width), getRandom(0, props.height), 2, 2);
    }
    ctx.fill();
  }

  captchaAnswer.value = result;
};

defineExpose({ answer: captchaAnswer, refresh: generate });
onMounted(generate);
</script>
