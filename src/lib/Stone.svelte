<script>
  import { createEventDispatcher } from 'svelte';
  import { T, useFrame } from '@threlte/core';
  import * as THREE from 'three';

  export let color = '#dfe2e8';
  export let roughness = 0.35;
  export let metalness = 0.1;
  export let scale = 1;
  export let distortion = 0.3;
  export let index = 0;
  export let offset = 0;
  export let interiorMix = 0;
  export let galleryMix = 0;

  const dispatch = createEventDispatcher();

  const geometry = new THREE.IcosahedronGeometry(0.9, 3);

  const noiseSize = 32;
  const data = new Uint8Array(noiseSize * noiseSize * 3);
  for (let i = 0; i < data.length; i += 3) {
    const value = Math.floor(120 + Math.random() * 120);
    data[i] = value;
    data[i + 1] = value;
    data[i + 2] = value;
  }

  const displacementMap = new THREE.DataTexture(data, noiseSize, noiseSize, THREE.RGBFormat);
  displacementMap.needsUpdate = true;
  displacementMap.wrapS = THREE.RepeatWrapping;
  displacementMap.wrapT = THREE.RepeatWrapping;

  const material = new THREE.MeshStandardMaterial({
    color,
    roughness,
    metalness,
    displacementMap,
    displacementScale: distortion,
    transparent: true,
    opacity: 0
  });

  let mesh;
  let floatOffset = Math.random() * Math.PI * 2;

  useFrame(({ clock }) => {
    if (!mesh) return;
    const time = clock.getElapsedTime();
    const floatY = Math.sin(time * 1.2 + floatOffset) * 0.18;

    const yPosition = (index - offset) * -4 + floatY;
    const baseOpacity = Math.max(interiorMix, galleryMix);

    mesh.position.set(0, 0.9 + yPosition, 0);
    mesh.rotation.y = time * 0.3 + index * 0.6;
    mesh.rotation.z = time * 0.2 + index * 0.4;
    mesh.scale.setScalar(scale);

    material.opacity = baseOpacity;
  });
</script>

<T.Mesh bind:this={mesh} geometry={geometry} material={material} on:click={() => dispatch('select')} />
