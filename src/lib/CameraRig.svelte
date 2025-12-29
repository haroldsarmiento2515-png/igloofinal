<script>
  import { onMount } from 'svelte';
  import { useTask } from '@threlte/core';
  import * as THREE from 'three';

  export let scrollProgress = 0;
  export let portalProgress = 0;

  const heroPosition = new THREE.Vector3(0, 1.8, 9);
  const interiorPosition = new THREE.Vector3(0, 1.4, 5.2);
  const galleryPosition = new THREE.Vector3(0, 1.2, 3.8);
  const portalPosition = new THREE.Vector3(0, 1.05, 1.6);

  const targetBase = new THREE.Vector3(0, 1.2, 0);

  const mouse = new THREE.Vector2(0, 0);
  const smoothMouse = new THREE.Vector2(0, 0);

  const clamp = (value, min = 0, max = 1) => Math.min(max, Math.max(min, value));

  onMount(() => {
    const handleMove = (event) => {
      mouse.x = (event.clientX / window.innerWidth - 0.5) * 2;
      mouse.y = (event.clientY / window.innerHeight - 0.5) * 2;
    };

    window.addEventListener('pointermove', handleMove);

    return () => {
      window.removeEventListener('pointermove', handleMove);
    };
  });

  useTask(({ camera }, delta) => {
    if (!camera) return;
    smoothMouse.lerp(mouse, 0.08);

    const heroMix = clamp(1 - scrollProgress * 2.4);
    const interiorMix = clamp((scrollProgress - 0.22) / 0.28);
    const galleryMix = clamp((scrollProgress - 0.52) / 0.48);

    const basePosition = heroPosition
      .clone()
      .lerp(interiorPosition, interiorMix)
      .lerp(galleryPosition, galleryMix);

    const portalMix = clamp(portalProgress);
    basePosition.lerp(portalPosition, portalMix);

    const offsetPosition = basePosition.clone().add(new THREE.Vector3(smoothMouse.x * 0.4, smoothMouse.y * 0.25, 0));

    camera.position.lerp(offsetPosition, 0.08);

    const lookAt = targetBase.clone();
    lookAt.x += smoothMouse.x * 0.5;
    lookAt.y += smoothMouse.y * 0.3;
    lookAt.z += heroMix * -0.4;

    camera.lookAt(lookAt);
    camera.updateProjectionMatrix();
  });
</script>
