<script setup>
// import HelloWorld from './components/HelloWorld.vue'
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { ColladaLoader } from 'three/examples/jsm/loaders/ColladaLoader';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';

  const globalYOffset = -2;
  const colors = [
    new THREE.Color(0xFA5151),
    new THREE.Color(0xFF9E00),
    new THREE.Color(0x19D152), 
    new THREE.Color(0x5DB7FF), 
    new THREE.Color(0x946BFF), 
    new THREE.Color(0xFFC500), 
    new THREE.Color(0x8072FF), 
    new THREE.Color(0x637CFF), 
  ];
  let figurine;
  let controls;
  let clock;
  let camera, scene, renderer;
  let spirals = [];

  setup();
  animate();

  function setup() {
    clock = new THREE.Clock();
    setupScene();
    setupCamera();
    // setupAxesHelper();
    setupLoader();
    setupLight();
    setupFloor();
    setupSpiral(0);
    setupSpiral(Math.PI);
    setupRenderer();
    // setupControls();
    setupListener();
  }

  function setupScene() {
    scene = new THREE.Scene();

    scene.fog = new THREE.Fog( 0x5DB7FF, 10, 100 );
  }

  function setupCamera() {
    // 初始化相机
    camera = new THREE.PerspectiveCamera(
      45,
      window.innerWidth / window.innerHeight,
      0.1, 
      1000
    );
    camera.position.set(10, 12, 10);
    camera.lookAt(0, 3, 0);
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
      figurine = collada.scene;
      figurine.castShadow = true;
      figurine.receiveShadow = true;
      figurine.position.set(0, globalYOffset, 0);
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
    // const ambientLight = new THREE.AmbientLight(0xffffff);
    const ambientLight = new THREE.AmbientLight(0x444444);
    scene.add(ambientLight);

    // const directionalLight = new THREE.DirectionalLight(0xffffff, 2.5);
    // directionalLight.position.set(1, 1, 0).normalize();
    // scene.add(directionalLight);

    const spotLight1 = createSpotlight( 0xffffff );
    const spotLight2 = createSpotlight( 0xffffff );
    const spotLight3 = createSpotlight( 0xffffff );
    spotLight1.position.set( 15, 20, 15 );
    spotLight2.position.set( 0, 20, -15 );
    spotLight3.position.set( -15, 20, 0 );
    scene.add( spotLight1, spotLight2, spotLight3 );
    
    // const lightHelper1 = new THREE.SpotLightHelper( spotLight1 );
    // const lightHelper2 = new THREE.SpotLightHelper( spotLight2 );
    // const lightHelper3 = new THREE.SpotLightHelper( spotLight3 );    
    // scene.add( lightHelper1, lightHelper2, lightHelper3 );
  }

  // 设置地板
  function setupFloor() {
    const matFloor = new THREE.MeshPhongMaterial( { color: 0xB0C4DE } );

    const geoFloor = new THREE.PlaneGeometry( 1000, 1000 );

    const mshFloor = new THREE.Mesh(geoFloor, matFloor);
    mshFloor.rotation.x = - Math.PI * 0.5;
    mshFloor.position.y = globalYOffset;
    mshFloor.receiveShadow = true;
    mshFloor.castShadow = true;
    scene.add(mshFloor);
  }

  // 初始化环境碎片
  function setupSpiral(startOffset) {
    const triangles = 250;

    const positions = new Float32Array(triangles * 3 * 3);
    const colors = new Float32Array(triangles * 3 * 3);
    const normals = new Float32Array(triangles * 3 * 3);

    let size = 1, size2 = size / 2; // individual triangle size

    const pA = new THREE.Vector3();
    const pB = new THREE.Vector3();
    const pC = new THREE.Vector3();

    const cb = new THREE.Vector3();
    const ab = new THREE.Vector3();

    let color = new THREE.Color();

    // positions 
    for ( let index = 0; index < triangles; index ++ ) {
      const t = index / 20;
      const i = index * 9;
      const progress = index / (triangles - 1);
      const ratio = 0.2 + 0.8 * progress;    // 三角形大小递增
      const d = size * ratio;
      const d2 = d / 2;

      const x = t * Math.cos(2 * t + startOffset);
      const y = t;
      const z = t * Math.sin(2 * t + startOffset);

      const ax = x + Math.random() * d - d2;
      const ay = y + Math.random() * d - d2;
      const az = z + Math.random() * d - d2;

      const bx = x + Math.random() * d - d2;
      const by = y + Math.random() * d - d2;
      const bz = z + Math.random() * d - d2;

      const cx = x + Math.random() * d - d2;
      const cy = y + Math.random() * d - d2;
      const cz = z + Math.random() * d - d2;

      positions[ i + 0 ] = ax;
      positions[ i + 1 ] = ay;
      positions[ i + 2 ] = az;

      positions[ i + 3 ] = bx;
      positions[ i + 4 ] = by;
      positions[ i + 5 ] = bz;

      positions[ i + 6 ] = cx;
      positions[ i + 7 ] = cy;
      positions[ i + 8 ] = cz;

      // flat face normals
      pA.set( ax, ay, az );
      pB.set( bx, by, bz );
      pC.set( cx, cy, cz );

      cb.subVectors( pC, pB );
      ab.subVectors( pA, pB );
      cb.cross( ab );

      cb.normalize();

      const nx = cb.x;
      const ny = cb.y;
      const nz = cb.z;

      normals[ i + 0 ] = nx;
      normals[ i + 1 ] = ny;
      normals[ i + 2 ] = nz;

      normals[ i + 3 ] = nx;
      normals[ i + 4 ] = ny;
      normals[ i + 5 ] = nz;

      normals[ i + 6 ] = nx;
      normals[ i + 7 ] = ny;
      normals[ i + 8 ] = nz;

      // Color
      const v = 0.5 + 0.5 * progress
      const vx = 117 / 255;
      const vy = 103 / 255;
      const vz = 1.0;

      // color.setRGB(v, v, v);
      // color.setHex(0x8072FF);
      color = randomColor();

      colors[ i + 0 ] = color.r;
      colors[ i + 1 ] = color.g;
      colors[ i + 2 ] = color.b;

      colors[ i + 3 ] = color.r;
      colors[ i + 4 ] = color.g;
      colors[ i + 5 ] = color.b;

      colors[ i + 6 ] = color.r;
      colors[ i + 7 ] = color.g;
      colors[ i + 8 ] = color.b;
    }

    let geometry = new THREE.BufferGeometry()
    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    geometry.setAttribute('normal', new THREE.BufferAttribute(normals, 3));
    geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

    geometry.computeBoundingSphere();

    let material = new THREE.MeshPhongMaterial({
      color: 0xaaaaaa, 
      specular: 0xffffff, 
      shininess: 250,
      side: THREE.DoubleSide,
      vertexColors: true
    });

    const spiral = new THREE.Mesh(geometry, material);
    spiral.position.set(0, -2 + globalYOffset, 0);
    spiral.castShadow = true;
    scene.add(spiral);
    spirals.push(spiral);
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
  }
  // const textureLoad = new THREE.TextureLoader();
  // textureLoad.load('TODO', (texture) => {
  //   texture.mapping = THREE.EquirectangularReflectionMapping;
  //   // 设置环境纹理
  //   scene.background = texture;
  //   scene.environment = texture;
  // });

  function setupListener() {
    window.addEventListener( 'resize', onWindowResize);
  }

  // 动画方法
  function animate() {
    requestAnimationFrame(animate);
    render();
  }

  // 渲染方法
  function render() {
    const delta = clock.getDelta();
    if (figurine !== undefined) {
      figurine.rotation.z += delta * 0.2;
    }
    for (let i = 0; i < spirals.length; i++) {
      const spiral = spirals[i];
      spiral.rotation.y += delta * 0.3;
    }
    renderer.render(scene, camera);
  }

  function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize( window.innerWidth, window.innerHeight );
  }

  function createSpotlight( color ) {
    const newObj = new THREE.SpotLight( color );
    newObj.castShadow = true;
    newObj.angle = 0.3;
    newObj.penumbra = 0.2;
    newObj.decay = 2;
    newObj.distance = 150;
    newObj.intensity = 1000;
    return newObj;
  }

  function randomColor() {
    const min = Math.ceil(0); // 向上取整
    const max = Math.floor(colors.length); // 向下取整
    const index = Math.floor(Math.random() * (max - min) + min);
    return colors[index];
  }
</script>

<template>
  <div>
    <canvas id="canvas"></canvas>
  </div>
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
