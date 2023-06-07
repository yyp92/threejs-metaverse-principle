<template>
  <div id="container"></div>
</template>

<script setup>
import * as THREE from "three";
import Stats from "three/examples/jsm/libs/stats.module.js";

import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

import { Octree } from "three/examples/jsm/math/Octree.js";
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper.js";

import { Capsule } from "three/examples/jsm/math/Capsule.js";

import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

import { reactive, onMounted, ref } from "vue";

onMounted(() => {
  const clock = new THREE.Clock();

  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x88ccee);
  // scene.fog = new THREE.Fog(0x88ccee, 0, 50);

  const camera = new THREE.PerspectiveCamera(
    70,
    window.innerWidth / window.innerHeight,
    0.001,
    1000
  );
  camera.position.set(0, 5, 10);

  const container = document.getElementById("container");

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.VSMShadowMap;
  renderer.outputEncoding = THREE.sRGBEncoding;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  container.appendChild(renderer.domElement);

  const stats = new Stats();
  stats.domElement.style.position = "absolute";
  stats.domElement.style.top = "0px";
  container.appendChild(stats.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);

  console.log(renderer.info);

  function animate() {
    let delta = clock.getDelta();

    stats.update();
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }

  // 创建一个平面
  const planeGeometry = new THREE.PlaneGeometry(20, 20, 1, 1);
  const planeMaterial = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
  });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  plane.receiveShadow = true;
  plane.rotation.x = -Math.PI / 2;
  scene.add(plane);

  // 形状和材质必须是相同的
  const geometry = new THREE.TorusKnotGeometry(1, 0.3, 100, 16);
  const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
  // const torusKnot = new THREE.Mesh(geometry, material);
  // torusKnot.position.set(0, 2, 0);
  // scene.add(torusKnot);

  // for (let i = 0; i < 1000; i++) {
  //   const torusKnot = new THREE.Mesh(geometry, material);
  //   torusKnot.position.set(
  //     Math.random() * 100 - 50,
  //     Math.random() * 100 - 50,
  //     Math.random() * 100 - 50
  //   );
  //   scene.add(torusKnot);
  // }

  const instanceMesh = new THREE.InstancedMesh(geometry, material, 10000);
  for (let i = 0; i < 10000; i++) {
    const position = new THREE.Vector3(
      Math.random() * 100 - 50,
      Math.random() * 100 - 50,
      Math.random() * 100 - 50
    );
    const rotation = new THREE.Euler(
      Math.random() * Math.PI * 2,
      Math.random() * Math.PI * 2,
      Math.random() * Math.PI * 2
    );
    // 四元数用于表示旋转
    const quaternion = new THREE.Quaternion().setFromEuler(rotation);
    const scale = new THREE.Vector3(
      Math.random() * 0.5 + 0.5,
      Math.random() * 0.5 + 0.5,
      Math.random() * 0.5 + 0.5
    );
    let matrix = new THREE.Matrix4();
    matrix.compose(position, quaternion, scale);

    instanceMesh.setMatrixAt(i, matrix);
  }

  scene.add(instanceMesh);

  animate();
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
#container {
  width: 100vw;
  height: 100vh;
}
</style>
