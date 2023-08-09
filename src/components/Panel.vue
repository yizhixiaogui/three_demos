<template>
  <div class="panel">
    <canvas class="can" id="fabric" ref="canvasRef"></canvas>
    <button class="btn" @click="sure('HouseholdItems_HQJ1005_3D_5')">正面贴图</button>
    <button class="btn" @click="sure('HouseholdItems_HQJ1005_3D_2')">反面贴图</button>
  </div>
</template>
<script setup>
import { fabric } from "fabric";
import { getCurrentInstance, onMounted, ref } from "vue";
import staticImg from "../assets/logo.png";

const context = getCurrentInstance() // 获取上下文
const $bus = context.appContext.config.globalProperties.$bus
const canvasRef = ref()
let canvas

const initCanvas = () => {
  canvas = new fabric.Canvas("fabric",{
    backgroundColor:"blue"
  });
  fabric.Image.fromURL(staticImg, (imgObj) => {
    canvas.add(imgObj);
  });
};

const setCanvasSize = ()=>{
  canvasRef.value.width = 400
  canvasRef.value.height = 400
  
}

const sure = (side)=>{
  let base64 = canvas.toDataURL("png")
  canvas.requestRenderAll()
  $bus.emit("upadteMaterial",{base64,side})
}

onMounted(() => {
  setCanvasSize()
  initCanvas();
});
</script>

<style scoped>
.panel {
  width: 600px;
  display: flex;
  align-items: center;
  flex-direction: column;
}
.panel .can {
  border: 1px solid #ff0000;
}
.panel .btn {
  margin-top: 20px;
  font-size: 20px;
}
</style>