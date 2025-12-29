<script>
  import { T, useTask } from '@threlte/core';
  import * as THREE from 'three';

  const geometry = new THREE.PlaneGeometry(32, 32, 120, 120);
  geometry.rotateX(-Math.PI / 2);

  const position = geometry.attributes.position;

  const baseHeights = new Float32Array(position.count);
  for (let i = 0; i < position.count; i += 1) {
    const x = position.getX(i);
    const z = position.getZ(i);
    const ridge = Math.sin(x * 0.25) * 0.8 + Math.cos(z * 0.18) * 0.6;
    const mound = Math.cos((x * x + z * z) * 0.02) * 0.9;
    baseHeights[i] = ridge + mound;
  }

  const material = new THREE.MeshStandardMaterial({
    color: '#cfd7e2',
    roughness: 0.95,
    metalness: 0.05
  });

  let elapsed = 0;

  useTask((_, delta) => {
    elapsed += delta;
    const time = elapsed;
    for (let i = 0; i < position.count; i += 1) {
      const x = position.getX(i);
      const z = position.getZ(i);
      const wind = Math.sin(time * 0.2 + x * 0.15 + z * 0.1) * 0.15;
      position.setY(i, baseHeights[i] + wind);
    }
    position.needsUpdate = true;
    geometry.computeVertexNormals();
  });
</script>

<T.Mesh geometry={geometry} material={material} receiveShadow />
