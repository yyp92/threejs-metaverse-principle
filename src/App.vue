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
import { LightningStorm } from "three/examples/jsm/objects/LightningStorm.js";

import { reactive, onMounted, ref } from "vue";
import fragmentShader from "./shader/fragmentShader.glsl";

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
  camera.position.set(0, 0, 50);

  const container = document.getElementById("container");

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.VSMShadowMap;
  renderer.outputEncoding = THREE.sRGBEncoding;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  container.appendChild(renderer.domElement);

  // 性能 - 渲染帧数
  const stats = new Stats();
  stats.domElement.style.position = "absolute";
  stats.domElement.style.top = "0px";
  container.appendChild(stats.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);

  console.log(renderer.info);

  function animate() {
    let delta = clock.getDelta();
    planeMaterial.uniforms.iTime.value += delta;
    stats.update();
    controls.update();
    storm.update(delta);
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }

  // 创建一个平面
  const planeGeometry = new THREE.PlaneGeometry(20, 20, 1, 1);
  const planeMaterial1 = new THREE.MeshBasicMaterial({
    color: 0xffffff,
    side: THREE.DoubleSide,
  });
  const plane2 = new THREE.Mesh(planeGeometry, planeMaterial1);
  plane2.rotation.x = -Math.PI / 2;
  scene.add(plane2);

  const planeMaterial = new THREE.ShaderMaterial({
    uniforms: {
      iTime: { value: 0 },
      iMouse: { value: new THREE.Vector2(0, 0) },
      iResolution: {
        value: new THREE.Vector2(1000, 1000),
      },
      rotation: { value: 0 },
    },
    vertexShader: `
    varying vec2 vUv;
    uniform float rotation;
      void main() {
        vUv = uv;
        vec4 mvPosition = modelViewMatrix * vec4( 0.0, 0.0, 0.0, 1.0 );
	      vec2 scale;
        scale.x = length( vec3( modelMatrix[ 0 ].x, modelMatrix[ 0 ].y, modelMatrix[ 0 ].z ) );
	      scale.y = length( vec3( modelMatrix[ 1 ].x, modelMatrix[ 1 ].y, modelMatrix[ 1 ].z ) );
        scale *= - mvPosition.z;
        vec2 alignedPosition = -position.xy * scale / mvPosition.z;
        vec2 rotatedPosition;
        rotatedPosition.x = cos( rotation ) * alignedPosition.x - sin( rotation ) * alignedPosition.y;
        rotatedPosition.y = sin( rotation ) * alignedPosition.x + cos( rotation ) * alignedPosition.y;
        mvPosition.xy += rotatedPosition;
        gl_Position = projectionMatrix * mvPosition;

        // vec4 viewPosition = viewMatrix * modelMatrix *vec4(position,1);
        // gl_Position = projectionMatrix * viewPosition;
        // gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1);
      }
    `,
    fragmentShader: fragmentShader,
    side: THREE.DoubleSide,
    transparent: true,
    depthWrite: false,
    // depthTest: false,
    blending: THREE.AdditiveBlending,
  });

  // const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  // plane.receiveShadow = true;
  // scene.add(plane);

  // 创建一个精灵
  const sprite = new THREE.Sprite(planeMaterial);
  sprite.renderOrder = 1;
  // sprite.scale.set(1, 1);
  sprite.position.set(0, 1, 0);
  scene.add(sprite);

  // 创建一个精灵
  const sprite2 = new THREE.Sprite(
    new THREE.SpriteMaterial({
      map: new THREE.TextureLoader().load("./textures/women.png"),
      color: 0xffffff,
      transparent: true,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
      depthTest: false,
    })
  );
  // scene.add(sprite2);
  sprite2.position.y = 1;
  sprite2.material.onBeforeCompile = (shader) => {
    console.log(shader.fragmentShader);
    console.log(shader.vertexShader);
  };
  console.log(sprite2);
  const storm = new LightningStorm({
    size: 100,
    minHeight: 90,
    maxHeight: 200,
    maxSlope: 0.6,
    maxLightnings: 8,

    onLightningDown: function (lightning) {
      console.log("lightning down", lightning);
    },
  });
  scene.add(storm);
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
