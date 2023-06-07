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
  camera.position.set(0, 0, 10);

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
  const planeGeometry = new THREE.PlaneGeometry(2, 2, 1, 1);
  const planeMaterial = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
    transparent: true,
    // blending: THREE.AdditiveBlending,
    depthWrite: false,
  });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  scene.add(plane);

  let alphaMap = new THREE.TextureLoader().load("./textures/chat_alpha.png");

  // 创建canvas对象
  const canvas = document.createElement("canvas");
  canvas.width = 1080;
  canvas.height = 1080;
  canvas.style.position = "absolute";
  canvas.style.top = "0px";
  canvas.style.left = "0px";
  canvas.style.zIndex = "1";
  canvas.style.transformOrigin = "0 0";
  canvas.style.transform = "scale(0.1)";
  const context = canvas.getContext("2d");

  var image = new Image();
  image.src = "./textures/chat.png";
  image.onload = function () {
    context.drawImage(image, 0, 0, 1080, 1080);
    context.textAlign = "center";
    context.textBaseline = "middle";
    context.font = "bold 100px Arial";
    context.fillStyle = "rgba(0,255,255,1)";
    context.fillText("Hello World", canvas.width / 2, canvas.height / 2);
    let texture = new THREE.CanvasTexture(canvas);
    plane.material.map = texture;
    plane.material.alphaMap = texture;
    plane.material.needsUpdate = true;
  };
  document.body.appendChild(canvas);

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
