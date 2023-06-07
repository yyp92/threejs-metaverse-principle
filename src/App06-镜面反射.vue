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
import { Reflector } from "three/examples/jsm/objects/Reflector.js";

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

    cubeCamera.update(renderer, scene);
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }

  const hdrLoader = new RGBELoader();
  hdrLoader.load("./hdr/023.hdr", (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    texture.format = THREE.RGBAFormat;
    scene.background = texture;
    scene.environment = texture;
    sphereMaterial.envMap = cubeRenderTarget.texture;
  });

  // console.log(sphereMaterial);
  // 创建球
  const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
  let sphereMaterial = new THREE.MeshPhysicalMaterial({
    color: 0xffffff,
    // 透光程度
    transparent: true,
    roughness: 0,
    metalness: 1,
  });

  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
  sphere.position.set(0, 0, 0);
  scene.add(sphere);

  // 创建立方体
  const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
  const boxMaterial = new THREE.MeshPhysicalMaterial({
    color: 0xffffff,
  });
  const box = new THREE.Mesh(boxGeometry, boxMaterial);
  box.position.set(3, 0, 0);
  scene.add(box);

  // 创建cubeTarget
  const cubeRenderTarget = new THREE.WebGLCubeRenderTarget(512);
  const cubeCamera = new THREE.CubeCamera(0.1, 1000, cubeRenderTarget);

  // 创建平面
  const planeGeometry = new THREE.PlaneGeometry(10, 10);
  // const planeMaterial = new THREE.MeshPhysicalMaterial({
  //   color: 0xffffff,
  // });
  // const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  const plane = new Reflector(planeGeometry, {
    textureWidth: 1024,
    textureHeight: 1024,
    color: 0xffffff,
  });
  plane.position.set(0, -1, 0);
  plane.rotation.x = -Math.PI / 2;
  scene.add(plane);

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
