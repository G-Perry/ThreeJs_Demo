<template>
  <div id="threeContainer" ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { onMounted } from 'vue';
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

let container, width, height; //threejs容器、宽、高
let scene, camera, renderer, controls, clock;//场景、相机、渲染器、轨道控制器、时钟
let model, skeleton, mixer;
let idleAction, walkAction, runAction;
let idleWeight, walkWeight, runWeight;
let actions, settings;
let keyStates = {}; //存储按键信息
function init() {
  container = document.getElementById('threeContainer')
  // console.dir(container);
  width = container.offsetWidth
  height = container.offsetHeight

  camera = new THREE.PerspectiveCamera(45, width / height, 1, 100);
  camera.rotation.order = 'YXZ';
  camera.position.set(1, 2, - 3);
  camera.lookAt(0, 1, 0);

  clock = new THREE.Clock();
  // 场景
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xa0a0a0);
  scene.fog = new THREE.Fog(0xa0a0a0, 10, 50);
  // 灯光
  const hemiLight = new THREE.HemisphereLight(0xffffff, 0x8d8d8d, 3);
  hemiLight.position.set(0, 20, 0);
  scene.add(hemiLight);
  const dirLight = new THREE.DirectionalLight(0xffffff, 3);
  dirLight.position.set(- 3, 10, - 10);
  dirLight.castShadow = true;
  dirLight.shadow.camera.top = 2;
  dirLight.shadow.camera.bottom = - 2;
  dirLight.shadow.camera.left = - 2;
  dirLight.shadow.camera.right = 2;
  dirLight.shadow.camera.near = 0.1;
  dirLight.shadow.camera.far = 40;
  scene.add(dirLight);
  // 地面
  const mesh = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), new THREE.MeshPhongMaterial({ color: 0xcbcbcb, depthWrite: false }));
  mesh.rotation.x = - Math.PI / 2;
  mesh.receiveShadow = true;
  scene.add(mesh);
  // 渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(width, height);
  // renderer.setAnimationLoop(animate);
  renderer.shadowMap.enabled = true;
  container.appendChild(renderer.domElement);
  // 添加坐标系
  const axesHelper = new THREE.AxesHelper(2); // 参数为坐标轴的长度
  scene.add(axesHelper);

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true; // 添加阻尼
  controls.dampingFactor = 0.1;
  controls.target.set(0, 1, 0);
  const loader = new GLTFLoader().setPath("./threeJsModel/");
  loader.load('Soldier.glb', function (gltf) {
    console.log(gltf);

    model = gltf.scene;
    scene.add(model);

    model.traverse(function (object) {

      if (object.isMesh) object.castShadow = true;

    });

    //

    skeleton = new THREE.SkeletonHelper(model);
    skeleton.visible = false;
    scene.add(skeleton);

    //

    // createPanel();


    //

    const animations = gltf.animations;

    mixer = new THREE.AnimationMixer(model);

    idleAction = mixer.clipAction(animations[0]);
    walkAction = mixer.clipAction(animations[3]);
    runAction = mixer.clipAction(animations[1]);

    actions = [idleAction, walkAction, runAction];
    setWeight(idleAction, 1.0);
    setWeight(walkAction, 0.0);
    setWeight(runAction, 0.0);

    actions.forEach(function (action) {
      action.play();
    });

    // runAction.play()
    renderer.setAnimationLoop(animate);
    // activateAllActions();

  });
}
let keyCodeSet = new Set(['KeyW', 'KeyA', 'KeyS', 'KeyD', 'ShiftLeft'])
function eventListener() {
  document.addEventListener('keydown', (event) => {
    if (keyCodeSet.has(event.code)) {
      keyStates[event.code] = true;
    }
  });
  document.addEventListener('keyup', (event) => {
    if (keyCodeSet.has(event.code)) {
      keyStates[event.code] = false;
    }
  });
  // container.addEventListener('mousedown', () => {
  //   document.body.requestPointerLock();
  //   // mouseTime = performance.now();
  // });
  // document.addEventListener('mouseup', () => {
  //   // if (document.pointerLockElement !== null) throwBall();
  // });
  // document.body.addEventListener('mousemove', (event) => {
  //   if (document.pointerLockElement === document.body) {
  //     camera.rotation.y -= event.movementX / 1500;
  //     camera.rotation.x -= event.movementY / 1500;
  //   }
  // });
}

const STATE = {
  IDLE: 'IDLE', // 停止状态
  WALK: 'WALK', // 走动状态
  IDLE_TO_WALK: 'IDLE_TO_WALK', // 停止过渡到走动
  WALK_TO_IDLE: 'WALK_TO_IDLE', // 走动过渡到停止
};

function keyStatesControls() {
  // console.log(
  //   keyStates
  // );
  if (keyStates['KeyW']) {
  }
  if (keyStates['KeyA']) {
  }
  if (keyStates['KeyS']) {
  }
  if (keyStates['KeyD']) {
  }
  if (keyStates['ShiftLeft']) {
  }
}

function setWeight(action, weight) {
  action.enabled = true;
  action.setEffectiveTimeScale(1);
  action.setEffectiveWeight(weight);
}

function animate() {
  let mixerUpdateDelta = clock.getDelta();
  if (mixer) {
    mixer.update(mixerUpdateDelta);
  }
  controls.update();
  renderer.render(scene, camera);
  keyStatesControls()
}

onMounted(() => {
  init()
  // animate()
  eventListener()
})
</script>

<style scoped>
#threeContainer {
  width: 100%;
  height: 100%;
}
</style>