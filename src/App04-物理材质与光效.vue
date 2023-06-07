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

import * as BufferGeometryUtils from "three/examples/jsm/utils/BufferGeometryUtils.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";

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

  animate();

  const hdrLoader = new RGBELoader();
  hdrLoader.load("./hdr/023.hdr", (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    texture.format = THREE.RGBAFormat;
    scene.background = texture;
    scene.environment = texture;
  });

  // 创建球
  const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
  const sphereMaterial = new THREE.MeshPhysicalMaterial({
    color: 0xffffff,
    // 透光程度
    transmission: 1,
    transparent: true,
    // 粗糙度
    roughness: 0,
    // 反色率
    reflectivity: 1,
    // 玻璃折射率
    ior: 2.33,
    // 彩虹的效果
    iridescenceIOR: 1.33,
  });
  // 支持彩虹效果的渲染（目前需要）
  sphereMaterial._iridescence = 1;
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
  sphere.position.set(0, 0, 0);
  scene.add(sphere);
  console.log(sphereMaterial);
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
