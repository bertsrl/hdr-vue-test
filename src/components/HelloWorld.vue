<template>
  <div id="parent-canvas" style="width: 100%; height: 100%">
    <canvas id="canvas" style="width: 100vw"></canvas>
  </div>
</template>
<script setup lang="ts">
import * as THREE from "three";
import { onMounted, ref, toRaw } from "vue";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";

const controlsRef = ref();
const rendererRef = ref();
const sceneRef = ref();
const cameraRef = ref();

const sizes = {
  width: window.innerWidth,
  height: window.innerHeight,
};

onMounted(() => {
  // Canvas
  const canvas = document.getElementById("canvas")!;

  const parentElement = document.getElementById("parent-canvas")!; // Replace with the actual ID or reference to the parent element

  const scene = new THREE.Scene();
  sceneRef.value = scene;

  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.01,
    1000
  );
  camera.position.z = 2;
  cameraRef.value = camera;
  // init renderer
  const renderer = new THREE.WebGLRenderer({
    alpha: true,
    canvas: canvas,
  });

  renderer.shadowMap.enabled = true;

  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 0.5;

  renderer.outputColorSpace = THREE.SRGBColorSpace;

  new RGBELoader().load("./textures/HDR/thatch_chapel_4k.hdr", (texture) => {
    const pmremGenerator = new THREE.PMREMGenerator(renderer);
    const envMap = pmremGenerator.fromEquirectangular(texture).texture;

    scene.background = texture;
    scene.environment = texture;

    // Adjust the intensity here
    envMap.colorSpace = THREE.SRGBColorSpace; // Ensure correct encoding
    scene.environment = envMap;

    texture.dispose();
    pmremGenerator.dispose();
  });

  rendererRef.value = renderer;

  resize(renderer, camera, sizes, parentElement);
  cursor(sizes, camera);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;

  controls.minDistance = 10;
  controls.maxDistance = 30;

  controlsRef.value = controls;

  const cubeMaterial = {
    clearcoat: 1.0,
    clearcoatRoughness: 0.1,
    metalness: 0.9,
    roughness: 0.5,
    color: 0x8418ca,
  };
  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshPhysicalMaterial(cubeMaterial);
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);

  animate();
});

function animate() {
  rendererRef.value.render(toRaw(sceneRef.value), toRaw(cameraRef.value));

  controlsRef.value.update();

  window.requestAnimationFrame(animate);
}

const pointer = new THREE.Vector2();
const raycaster = new THREE.Raycaster();

function cursor(
  sizes: { width: number; height: number },
  camera?: THREE.Camera
) {
  window.addEventListener("mousemove", (event) => {
    pointer.x = event.clientX / sizes.width - 0.5;
    pointer.y = -(event.clientY / sizes.height) + 0.5;

    raycaster.setFromCamera(pointer, camera!);
  });
}

function resize(
  renderer: THREE.WebGLRenderer,
  camera: THREE.PerspectiveCamera,
  sizes: { width: number; height: number },
  parentElement: HTMLElement
) {
  const updateRendererSize = () => {
    const rect = parentElement.getBoundingClientRect();

    // Update sizes
    sizes.width = rect.width;
    sizes.height = rect.height;

    // Update camera
    camera.aspect = sizes.width / sizes.height;
    camera.updateProjectionMatrix();

    // Update renderer
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  };
  // Initial call to set sizes
  updateRendererSize();

  // Update on window resize
  window.addEventListener("resize", updateRendererSize);
}
</script>
<style lang=""></style>
