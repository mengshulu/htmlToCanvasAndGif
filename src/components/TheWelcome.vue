<script setup>
import '../js/gif.js';
import html2canvas from 'html2canvas';
import { ref } from 'vue';

const refCanvasList = ref([]);
const refImg = ref(null);
const itemNum = ref(5);
const nowNum = ref(1);

function timeout(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

const clickToCanvas = async () => {
  console.log(refImg.value.children);
    var gif = new GIF({
    workers: 2,
    quality: 10,
    width: 100,
    height: 100,
    workerScript: './src/js/gif.worker.js'
  });

  for(let i=0; i<refCanvasList.value.length;i++) {
    console.log(refCanvasList.value[i]);

    html2canvas(refCanvasList.value[i]).then((canvas) => {
      refImg.value.appendChild(canvas);
    });
    nowNum.value = nowNum.value+1;
    console.log('nowNum.value: ', nowNum.value);
    await timeout(1000);
    gif.addFrame(refImg.value.children[i]);
  }
  nowNum.value = 1;
  console.log('finished nowNum.value: ', nowNum.value);

  gif.on('finished', function(blob) {
    console.log('blob: ', blob);

    saveAs(URL.createObjectURL(blob), 'test-gif.gif');
    
    window.open(URL.createObjectURL(blob));
  });

  

  gif.render();
};

const saveAs = (uri, filename) => {
    let link = document.createElement('a');
    if (typeof link.download === 'string') {
        link.href = uri;
        link.download = filename;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    } else {
        window.open(uri);
    }
};
</script>

<template>
  <div 
    v-for="item in itemNum" 
    ref="refCanvasList"
    :class="{ 'active': item === nowNum}"
    class="canvas"
  >canvas {{ item }}</div>
  <button @click="clickToCanvas">toCanvas</button>
  <div ref="refImg"></div>
</template>

<style lang="scss">
  .canvas {
    display: none;
    width: 100px;
    height: 100px;
    background-color: #aaa;
    border: 1px solid #000;

    &.active {
      display: block;
    }
  }
</style>
