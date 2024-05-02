<script setup>
// import HelloWorld from './components/HelloWorld.vue'
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { ColladaLoader } from 'three/examples/jsm/loaders/ColladaLoader';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';


  let figurine;
  let controls;
  let clock;
  let camera, scene, renderer;

  setup();
  animate();

  function setup() {
    clock = new THREE.Clock();
    scene = new THREE.Scene();
    setupCamera();
    setupAxesHelper();
    setupLoader();
    setupLight();
    setupRenderer();
    setupControls();
  }

  function setupCamera() {
    // 初始化相机
    camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1, 
      100
    );
    camera.position.set(0, 0, 8);
    camera.updateProjectionMatrix();
  }

  function setupAxesHelper() {
    const axesHelper = new THREE.AxesHelper(5);
    scene.add(axesHelper);
  }

  // 初始化加载器
  function setupLoader() {
    const loadingManager = new THREE.LoadingManager(() => {
      scene.add(figurine);
    });
    // // 加载模型
    const colladaLoader = new ColladaLoader(loadingManager);
    colladaLoader.load('./model/elf/elf.dae', (collada) => {
      console.log("loadded");
      figurine = collada.scene;
    });

    // TODO: @baocq
    // const loader = new GLTFLoader();
    // const dracoLoader = new DRACOLoader();
    // dracoLoader.setDecoderPath('./draco/');
    // loader.setDRACOLoader(dracoLoader);
    // loader.load('./model/island2.glb', (gltf) => {
    //   console.log("load success");
    //   scene.add(gltf.scene);
    // }, (xhr) => {
    //   console.log('load progress: ' + ( xhr.loaded / xhr.total * 100 ) + '%');
    // }, (error) => {
    //   console.error( "load error: ", error );
    // });
  }
  
  // 设置灯光
  function setupLight() {
    const ambientLight = new THREE.AmbientLight(0xffffff);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 2.5);
    directionalLight.position.set(1, 1, 0).normalize();
    scene.add(directionalLight);
  }
  // 初始化渲染器
  function setupRenderer() {
    renderer = new THREE.WebGLRenderer({
      antialias: true,
      logarithmicDepthBuffer: true,
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    // 色调映射
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.toneMappingExposure = 1;
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);
  }

  function setupControls() {
    // 初始化控制器
    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    // controls.autoRotate = true;
  }
  // const textureLoad = new THREE.TextureLoader();
  // textureLoad.load('TODO', (texture) => {
  //   texture.mapping = THREE.EquirectangularReflectionMapping;
  //   // 设置环境纹理
  //   scene.background = texture;
  //   scene.environment = texture;
  // });

  // 动画方法
  function animate() {
    requestAnimationFrame(animate);
    render();
  }

  // 渲染方法
  function render() {
    // console.log("render");
    const delta = clock.getDelta();
    if (figurine !== undefined) {
      figurine.rotation.z += delta * 0.5;
    }
    renderer.render(scene, camera);
    // controls.update();
  }
</script>

<template>
  <div></div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

canvas {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}
</style>
