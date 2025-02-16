<template>
  <div id="threeContainer" ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { onMounted } from 'vue';
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

let width, height, scene, camera, renderer, controls, model, skeleton, mixer
let idleAction, walkAction, runAction;
let idleWeight, walkWeight, runWeight;
let actions, settings;

function init() {
  let container = document.getElementById('threeContainer')
  // console.dir(container);
  width = container.offsetWidth
  height = container.offsetHeight

  camera = new THREE.PerspectiveCamera(45, width / height, 1, 100);
  camera.position.set(1, 2, - 3);
  camera.lookAt(0, 1, 0);

  // clock = new THREE.Clock();
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
  renderer.shadowMap.enabled = true;
  container.appendChild(renderer.domElement);
  // 添加坐标系
  const axesHelper = new THREE.AxesHelper(20); // 参数为坐标轴的长度
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

    // activateAllActions();

    // animate();

  });
}
function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}


onMounted(() => {
  init()
  animate()
})
</script>

<style scoped>
#threeContainer {
  width: 100%;
  height: 100%;
}
</style>