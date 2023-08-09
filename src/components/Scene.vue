<template>
  <div class="main" ref="threeRef"></div>
</template>

<script setup>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { getCurrentInstance, onMounted, ref } from "vue";

const context = getCurrentInstance(); // 获取上下文
const $bus = context.appContext.config.globalProperties.$bus;
const threeRef = ref();
let scene, camera, lights, renderer, control, textureLoad;

// 创建相机
const initCamera = (element) => {
  camera = new THREE.PerspectiveCamera(
    50,
    element.value.offsetWidth / element.value.offsetHeight,
    0.01,
    1000
  );
  camera.position.set(0, 0, 3);
  camera.layers.enableAll();
  return camera;
};

// 创建场景
const initScene = () => {
  scene = new THREE.Scene();
  // 添加场景灯光
  for (let light of lights) {
    scene.add(light);
  }
  // 添加环境光
  const ambientLight = new THREE.AmbientLight(0x828f99, 6);
  scene.add(ambientLight);
  // 添加坐标轴
  // const axesHelper = new THREE.AxesHelper(10)
  // scene.add(axesHelper)
  return scene;
};

// 创建灯光组
const initLight = () => {
  let directionalLights = [];
  const loadLight = () => {
    const LIGHTS = [
      [100, 100, 100],
      [-100, 100, 100],
      [100, -100, 100],
      [100, 100, -100],
    ];
    LIGHTS.forEach(([x, y, z]) => {
      const directionalLight = new THREE.DirectionalLight(0xffffff, 5);
      directionalLight.position.set(x, y, z);
      directionalLight.castShadow = true; // 开启投影
      directionalLight.shadow.camera.far = 300;
      directionalLight.shadow.mapSize.set(2048, 2048);
      directionalLight.shadow.normalBias = 0.05;
      directionalLight.position.set(15, 30, 15);
      directionalLights.push(directionalLight);
    });
  };
  loadLight();
  return directionalLights;
};

// 创建渲染器
const initRenderer = (element) => {
  renderer = new THREE.WebGLRenderer({
    // 如果想保存three.js canvas画布上的信息，注意设置preserveDrawingBuffer
    preserveDrawingBuffer: true,
    antialias: true, // 抗锯齿开启
  });
  renderer.shadowMap.enabled = true;
  // renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 可根据需要选择阴影类型
  renderer.setClearColor("#999");
  // renderer.toneMappingExposure = 1
  renderer.physicallyCorrectLights = true; // 采用物理上正确的光照模式
  // renderer.toneMapping = THREE.CineonToneMapping
  renderer.outputEncoding = THREE.sRGBEncoding;

  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(element.value.offsetWidth, element.value.offsetHeight);
  renderer.localClippingEnabled = true;
  element.value.appendChild(renderer.domElement);

  // ------------------------

  renderer.toneMapping = THREE.ACESFilmicToneMapping; // 设置toneMapping为ACES Filmic
  renderer.outputEncoding = THREE.sRGBEncoding; // 设置outputEncoding为sRGBEncoding
  // renderer.toneMappingExposure = -0.39; // 设置toneMappingExposure为-0.39
  renderer.toneMappingExposure = 1; // 设置toneMappingExposure为-0.39
  return renderer;
};

// 帧刷新
const animate = () => {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
  control.update();
};

// 创建控制器
const initControl = () => {
  const c = new OrbitControls(camera, renderer.domElement);
  c.enableDamping = true;
  c.target = new THREE.Vector3(0, 0, 0);
  return c;
};

// 创建纹理加载器
const initTextureLoad = () => {
  textureLoad = new THREE.TextureLoader();
  return textureLoad;
};

// 创建窗口监听器
const initEventListener = (element, Trenderer) => {
  window.addEventListener(
    "resize",
    () => {
      camera.aspect = element.value.offsetWidth / element.value.offsetHeight;
      camera.updateProjectionMatrix();
      Trenderer.setSize(element.value.offsetWidth, element.value.offsetHeight);
    },
    false
  );
};

// 初始化场景元素
const initSceneChildren = () => {
  // 加载静态模型
  const loadStaticModel = () => {
    const loader = new GLTFLoader();
    const dracolLoader = new DRACOLoader();
    dracolLoader.setDecoderPath(`/static/draco/`);
    loader.setDRACOLoader(dracolLoader);
    const url = `/model/3d-file.glb`;
    loader.load(url, (gltf) => {
      let modelObj = gltf.scene.children[0].clone();
      modelObj.name = "pillow";
      modelObj.position.set(0, 0, 0);
      modelObj.castShadow = true;
      modelObj.receiveShadow = true;
      scene.add(modelObj);
    });
  };
  loadStaticModel();
};

// 更新贴图
const updateMaterial = ({ base64, side }) => {
  const model = scene.getObjectByName("pillow").getObjectByName(side);
  const box = new THREE.Box3().setFromObject(model)
  const center = box.getCenter(new THREE.Vector3())
  const size = box.getSize(new THREE.Vector3())
  const texture = textureLoad.load(base64);
  // 设置阵列模式
  texture.wrapS = texture.wrapT = THREE.RepeatWrapping;
  // UV 两个方向纹理重复数量
  texture.repeat.set(1,1)
  // 设置纹理旋转角度
  // texture.rotation = Math.PI/4;
  // 设置纹理的旋转中心，默认(0,0)
  // texture.center.set(0.5,0.5);
  // 设置纹理缩放
  // 创建贴图材质
  const material = new THREE.MeshBasicMaterial({
    map: texture,
    side: THREE.DoubleSide,
  });

  model.material.map = texture;
  model.material.needsUpdate = true;

  // 贴图自适应
};

// 挂载
onMounted(() => {
  $bus.on("upadteMaterial", (config) => updateMaterial(config));

  lights = initLight();
  scene = initScene();
  camera = initCamera(threeRef);
  renderer = initRenderer(threeRef);
  control = initControl();
  textureLoad = initTextureLoad();

  initEventListener(threeRef, renderer);
  animate();
  initSceneChildren();
});
</script>
<style>
.main {
  width: calc(100vw - 600px);
  height: 100vh;
}
</style>