<script>
  import { T, useTask } from '@threlte/core';
  import * as THREE from 'three';

  export let heroMix = 1;

  const count = 1200;
  const stride = 3;

  const array = new Float32Array(count * stride);
  const interleaved = new THREE.InterleavedBuffer(array, stride);
  const positions = new THREE.InterleavedBufferAttribute(interleaved, 3, 0);

  const basePositions = [];

  for (let i = 0; i < count; i += 1) {
    const x = (Math.random() - 0.5) * 22;
    const y = Math.random() * 10 + 0.5;
    const z = (Math.random() - 0.5) * 18;
    positions.setXYZ(i, x, y, z);
    basePositions.push({ x, y, z, offset: Math.random() * Math.PI * 2 });
  }

  const geometry = new THREE.BufferGeometry();
  geometry.setAttribute('position', positions);

  const material = new THREE.PointsMaterial({
    size: 0.08,
    color: '#f8fbff',
    transparent: true,
    opacity: 0.9,
    depthWrite: false,
    depthTest: false
  });

  let elapsed = 0;

  useTask((_, delta) => {
    elapsed += delta;
    const time = elapsed;

    for (let i = 0; i < count; i += 1) {
      const base = basePositions[i];
      const wobble = Math.sin(time * 0.8 + base.offset) * 0.18;
      const drift = (time * 0.06 + base.offset) % 8;
      positions.setXYZ(
        i,
        base.x + wobble,
        base.y - drift,
        base.z + Math.cos(time * 0.6 + base.offset) * 0.12
      );
    }

    positions.needsUpdate = true;
    material.opacity = 0.9 * heroMix;
  });
</script>

<T.Points geometry={geometry} material={material} />
