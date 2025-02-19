<template>
  <div id="threeContainer" ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { onMounted } from 'vue';
import { Capsule } from 'three/addons/math/Capsule.js';
import { Octree } from 'three/addons/math/Octree.js';

// import { OrbitControls } from "three/addons/controls/OrbitControls.js";

let container, width, height; //threejs容器、宽、高
let scene, camera, renderer, controls, clock;//场景、相机、渲染器、轨道控制器、时钟
let model, skeleton, mixer; // 模型、骨骼、动画混合器
let idleAction, walkAction, runAction; // 动画动作
// let idleWeight, walkWeight, runWeight;
// let actions, settings; // 动画动作数组
let keyStates = {}; //存储按键信息
let activeAction, previousAction; // 当前动作和上一个动作
function init() {
  container = document.getElementById('threeContainer')
  // console.dir(container);
  width = container.offsetWidth
  height = container.offsetHeight

  camera = new THREE.PerspectiveCamera(45, width / height, 1, 100);
  camera.rotation.order = 'YXZ';
  camera.position.set(1, 2, -3);
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
  worldOctree.fromGraphNode(mesh)
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

  // controls = new OrbitControls(camera, renderer.domElement);
  // controls.enableDamping = true; // 添加阻尼
  // controls.dampingFactor = 0.1;
  // controls.target.set(0, 1, 0);
  const loader = new GLTFLoader().setPath("./threeJsModel/");
  loader.load('bot_action.glb', function (gltf) {
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
    walkAction = mixer.clipAction(animations[2]);
    runAction = mixer.clipAction(animations[1]);

    // actions = [idleAction, walkAction, runAction];
    // setWeight(idleAction, 1.0);
    // setWeight(walkAction, 0.0);
    // setWeight(runAction, 0.0);

    // actions.forEach(function (action) {
    //   action.play();
    // });
    activeAction = idleAction;
    previousAction = idleAction;
    activeAction.play();

    // runAction.play()
    // renderer.setAnimationLoop(animate);
    // activateAllActions();
    // 在模型加载成功后调用
    renderer.setAnimationLoop(null); // 先取消原有循环
    requestAnimationFrame(animate);
  });
}

// let keyCodeSet = new Set(['KeyW', 'KeyA', 'KeyS', 'KeyD', 'ShiftLeft'])
// function eventListener() {
//   document.addEventListener('keydown', (event) => {
//     if (keyCodeSet.has(event.code)) {
//       keyStates[event.code] = true;
//     }
//   });
//   document.addEventListener('keyup', (event) => {
//     if (keyCodeSet.has(event.code)) {
//       keyStates[event.code] = false;
//     }
//   });
//   // container.addEventListener('mousedown', () => {
//   //   document.body.requestPointerLock();
//   //   // mouseTime = performance.now();
//   // });
//   // // document.addEventListener('mouseup', () => {
//   // //   // if (document.pointerLockElement !== null) throwBall();
//   // // });
//   // document.body.addEventListener('mousemove', (event) => {
//   //   if (document.pointerLockElement === document.body) {
//   //     camera.rotation.y -= event.movementX / 1500;
//   //     camera.rotation.x -= event.movementY / 1500;
//   //   }
//   // });
// }
function throttle(callback, delay = 100) {
  let lastCall = 0;
  return (...args) => {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      callback(...args);
    }
  };
}
const keyCodeSet = {
  KeyW: true,
  KeyA: true,
  KeyS: true,
  KeyD: true,
  ShiftLeft: true,
  Space: true,
};

function eventListener() {
  const handleKeyDown = throttle((event) => {
    if (keyCodeSet[event.code]) keyStates[event.code] = true;
  }, 100);

  document.addEventListener('keydown', handleKeyDown);
  document.addEventListener('keyup', (event) => {
    if (keyCodeSet[event.code]) keyStates[event.code] = false;
  });
  container.addEventListener('mousedown', () => {
    document.body.requestPointerLock();
    // mouseTime = performance.now();
  });
  // document.addEventListener('mouseup', () => {
  //   // if (document.pointerLockElement !== null) throwBall();
  // });
  document.body.addEventListener('mousemove', (event) => {
    if (document.pointerLockElement === document.body) {
      camera.rotation.y -= event.movementX / 1500;
      camera.rotation.x -= event.movementY / 1500;
    }
  });
}

const STATE = {
  IDLE: 'IDLE', // 停止状态
  WALK: 'WALK', // 走动状态
  RUN: 'RUN', // 奔跑状态
  // IDLE_TO_WALK: 'IDLE_TO_WALK', // 停止过渡到走动
  // WALK_TO_IDLE: 'WALK_TO_IDLE', // 走动过渡到停止
  // WALK_TO_RUN: 'WALK_TO_RUN', // 走动过渡到奔跑
  // RUN_TO_WALK: 'RUN_TO_WALK', // 奔跑过渡到走动
  // RUN_TO_IDLE: 'RUN_TO_IDLE', // 奔跑过渡到走动
};
let currentState = STATE.IDLE; // 当前状态
// let targetState = STATE.IDLE; // 目标状态
// let transitionTime = 0.5; // 过渡时间
// let transitionProgress = 0; // 过渡进度

function updateState(deltaTime) {
  let isMoving = keyStates['KeyW'] || keyStates['KeyA'] || keyStates['KeyS'] || keyStates['KeyD'];
  let isRunning = keyStates['ShiftLeft'] && isMoving
  // if (currentState === STATE.IDLE) {
  //   if (isMoving) {
  //     targetState = STATE.WALK;
  //     currentState = STATE.IDLE_TO_WALK;
  //     transitionProgress = 0;
  //   }
  // }
  // if (currentState === STATE.WALK) {
  //   if (isRunning) {
  //     targetState = STATE.RUN;
  //     currentState = STATE.WALK_TO_RUN;
  //     transitionProgress = 0;
  //   }
  //   if (!isMoving) {
  //     targetState = STATE.IDLE;
  //     currentState = STATE.WALK_TO_IDLE;
  //     transitionProgress = 0;
  //   }
  // }
  // if (currentState === STATE.RUN) {
  //   if (!isRunning) {
  //     targetState = STATE.WALK;
  //     currentState = STATE.RUN_TO_WALK;
  //     transitionProgress = 0;
  //   }
  //   if (!isMoving) {
  //     targetState = STATE.IDLE;
  //     currentState = STATE.RUN_TO_IDLE;
  //     transitionProgress = 0;
  //   }
  // }
  // 如果状态未变化，直接返回
  // if (
  //   (currentState === STATE.RUN && isMoving && isRunning) ||
  //   (currentState === STATE.WALK && isMoving && !isRunning) ||
  //   (currentState === STATE.IDLE && !isMoving)
  // ) {
  //   return;
  // }

  // 更新状态
  if (isMoving && isRunning) {
    // currentState = STATE.RUN;
    switchAction(runAction);
  } else if (isMoving) {
    // currentState = STATE.WALK;
    switchAction(walkAction);
  } else {
    // currentState = STATE.IDLE;
    switchAction(idleAction, 0.5);
  }

  const speedDelta = deltaTime * (playerOnFloor ? 25 : 8);
  if (keyStates['KeyW']) {
    playerVelocity.add(getForwardVector().multiplyScalar(speedDelta));
  }

  if (keyStates['KeyS']) {
    playerVelocity.add(getForwardVector().multiplyScalar(- speedDelta));
  }

  if (keyStates['KeyA']) {
    playerVelocity.add(getSideVector().multiplyScalar(- speedDelta));
  }

  if (keyStates['KeyD']) {
    playerVelocity.add(getSideVector().multiplyScalar(speedDelta));
  }
  if (playerOnFloor) {
    if (keyStates['Space']) {
      playerVelocity.y = 10;
    }
  }
}

// // 更新动画权重
// function updateAnimationWeights(delta) {
//   transitionProgress += delta / transitionTime;
//   // console.log(currentState);

//   if (currentState === STATE.IDLE_TO_WALK) {
//     prepareCrossFade(idleAction, walkAction, 0.5);
//     if (transitionProgress >= 1) {
//       currentState = STATE.WALK;
//       transitionProgress = 1;
//     }
//   } else if (currentState === STATE.WALK_TO_IDLE) {
//     prepareCrossFade(walkAction, idleAction, 0.5);
//     if (transitionProgress >= 1) {
//       currentState = STATE.IDLE;
//       transitionProgress = 1;
//     }
//   } else if (currentState === STATE.WALK_TO_RUN) {
//     prepareCrossFade(walkAction, runAction, 0.1);
//     if (transitionProgress >= 1) {
//       currentState = STATE.RUN;
//       transitionProgress = 1;
//     }
//   } else if (currentState === STATE.RUN_TO_WALK) {
//     prepareCrossFade(runAction, walkAction, 0.5);
//     if (transitionProgress >= 1) {
//       currentState = STATE.WALK;
//       transitionProgress = 1;
//     }
//   }
// }


// function prepareCrossFade(startAction, endAction, defaultDuration) {
//   // const duration = setCrossFadeDuration(defaultDuration);
//   // singleStepMode = false;
//   let duration = defaultDuration
//   // actions.forEach(function (action) {
//   //   action.paused = false;
//   // });
//   // if (startAction === idleAction) {
//   executeCrossFade(startAction, endAction, duration);
//   // } else {
//   //   synchronizeCrossFade(startAction, endAction, duration);
//   // }
// }
// function switchAction(newAction, fadeDuration = 0.4) {
//   if (activeAction !== newAction) {
//     previousAction = activeAction;
//     activeAction = newAction;
//     // 仅在动画未播放时重置
//     if (!activeAction.isRunning()) {
//       activeAction.reset();
//     }
//     // 淡出上一个动作
//     previousAction.fadeOut(fadeDuration);
//     // 淡入新动作
//     activeAction.fadeIn(fadeDuration).play();
//   }
// }
// 切换动作
function switchAction(newAction, fadeDuration = 0.4) {
  if (activeAction !== newAction) {
    // 淡出当前动作
    activeAction.fadeOut(fadeDuration);
    // 淡入新动作
    newAction.reset().fadeIn(fadeDuration).play();
    // 更新当前动作
    activeAction = newAction;
  }
}
// function switchAction(newAction, fadeDuration = 0.4) {
//   if (activeAction !== newAction) {
//     previousAction = activeAction;
//     activeAction = newAction;

//     // 重置新动作的时间轴
//     activeAction.reset();

//     // 确保上一个动作的权重为1
//     previousAction.setEffectiveWeight(1);
//     // 确保新动作的权重为1
//     activeAction.setEffectiveWeight(1);

//     // 执行过渡
//     previousAction.crossFadeTo(activeAction, fadeDuration, true);
//     activeAction.play();
//   }
// }
// function executeCrossFade(startAction, endAction, duration) {
//   // console.log(1111);

//   setWeight(startAction, 1);
//   endAction.time = 0;
//   // startAction.crossFadeTo(endAction, duration, true);
//   endAction.fadeIn(duration)
//   startAction.fadeOut(duration)
// }
// function synchronizeCrossFade(startAction, endAction, duration) {
//   mixer.addEventListener('loop', onLoopFinished);
//   function onLoopFinished(event) {
//     if (event.action === startAction) {
//       mixer.removeEventListener('loop', onLoopFinished);
//       executeCrossFade(startAction, endAction, duration);
//     }
//   }
// }

// function setWeight(action, weight) {
//   action.enabled = true;
//   action.setEffectiveTimeScale(1);
//   action.setEffectiveWeight(weight);
// }
let playerOnFloor = false;
let playerVelocity = new THREE.Vector3();
let playerDirection = new THREE.Vector3();
let playerCollider = new Capsule(new THREE.Vector3(0, 0.35, 0), new THREE.Vector3(0, 2, 0), 0.35);
let worldOctree = new Octree();
let GRAVITY = 30;
function updatePlayer(deltaTime) {

  let damping = Math.exp(- 4 * deltaTime) - 1;

  if (!playerOnFloor) {
    playerVelocity.y -= GRAVITY * deltaTime;
    damping *= 0.1;
  }

  playerVelocity.addScaledVector(playerVelocity, damping);

  const deltaPosition = playerVelocity.clone().multiplyScalar(deltaTime);
  playerCollider.translate(deltaPosition);

  playerCollisions();

  camera.position.copy(playerCollider.end);
  // model.position.copy(playerCollider.end)
}
function playerCollisions() {

  const result = worldOctree.capsuleIntersect(playerCollider);

  playerOnFloor = false;

  if (result) {

    playerOnFloor = result.normal.y > 0;

    if (!playerOnFloor) {

      playerVelocity.addScaledVector(result.normal, - result.normal.dot(playerVelocity));

    }

    if (result.depth >= 1e-10) {

      playerCollider.translate(result.normal.multiplyScalar(result.depth));

    }

  }

}

function getForwardVector() {

  camera.getWorldDirection(playerDirection);
  playerDirection.y = 0;
  playerDirection.normalize();

  return playerDirection;

}
function getSideVector() {

  camera.getWorldDirection(playerDirection);
  playerDirection.y = 0;
  playerDirection.normalize();
  playerDirection.cross(camera.up);

  return playerDirection;

}



function animate() {
  // console.log(
  //   idleAction.getEffectiveWeight(),
  //   walkAction.getEffectiveWeight(),
  //   runAction.getEffectiveWeight()
  // );
  // console.log(camera.position);
  // console.log(camera.rotation);


  let delta = clock.getDelta();
  if (mixer) mixer.update(delta);
  updateState(delta)
  // updateAnimationWeights(mixerUpdateDelta)
  updatePlayer(delta)
  // controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
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