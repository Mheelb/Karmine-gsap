<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { SVGLoader } from "three/examples/jsm/loaders/SVGLoader.js";
import { gsap } from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { ScrollSmoother } from "gsap-trial/ScrollSmoother";

gsap.registerPlugin(ScrollTrigger, ScrollSmoother);

const canvasRef = ref<HTMLDivElement | null>(null);

onMounted(() => {
  if (!canvasRef.value) return;

  ScrollSmoother.create({
    wrapper: ".scroll-container",
    content: ".scroll-content",
    smooth: 1.5,
    effects: true,
  });

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / document.body.scrollHeight,
    0.1,
    1000
  );
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, document.body.scrollHeight);
  renderer.setClearColor(0x000000);
  canvasRef.value.appendChild(renderer.domElement);

  const loader = new SVGLoader();
  loader.load("/Karmine_Corp_logo.svg", (data) => {
    const paths = data.paths;
    const group = new THREE.Group();

    paths.forEach((path) => {
      const shapes = SVGLoader.createShapes(path);
      shapes.forEach((shape) => {
        const extrudeSettings = {
          depth: 200,
          bevelEnabled: true,
          bevelThickness: 1,
          bevelSize: 0.5,
        };
        const geometry = new THREE.ExtrudeGeometry(shape, extrudeSettings);
        const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
        const mesh = new THREE.Mesh(geometry, material);
        group.add(mesh);
      });
    });

    group.scale.set(0.01, -0.01, 0.01);
    group.position.set(0, 20, 0);
    scene.add(group);

    camera.position.z = 50;

    gsap.to(group.position, {
      y: -400,
      z: 1000,
      scrollTrigger: {
        trigger: canvasRef.value,
        start: "top top",
        end: "bottom top",
        scrub: true,
        markers: true,
      },
    });
    gsap.to(group.rotation, {
      y: Math.PI * 40,
      scrollTrigger: {
        trigger: canvasRef.value,
        start: "top top",
        end: "bottom top",
        scrub: true,
        markers: true,
      },
    });

    animate();
  });

  function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  }
});
</script>

<template>
  <div class="scroll-container">
    <div class="scroll-content">
      <div ref="canvasRef" class="scene"></div>
    </div>
  </div>
</template>

<style scoped>
.scroll-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  position: relative;
}

.scroll-content {
  position: absolute;
  width: 100%;
  height: 200vh;
}

.scene {
  width: 100vw;
  height: 3300vh;
  display: block;
}
</style>