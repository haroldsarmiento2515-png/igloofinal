<script>
  import { T } from '@threlte/core';
  import * as THREE from 'three';

  const geometry = new THREE.PlaneGeometry(40, 16, 120, 40);
  const position = geometry.attributes.position;

  for (let i = 0; i < position.count; i += 1) {
    const x = position.getX(i);
    const y = position.getY(i);
    const ridge = Math.sin(x * 0.18) * 2.2 + Math.cos(x * 0.05) * 1.6;
    const swell = Math.sin((x + y) * 0.08) * 0.6;
    position.setY(i, y + ridge + swell);
  }

  geometry.computeVertexNormals();

  const material = new THREE.MeshStandardMaterial({
    color: '#c7d0db',
    roughness: 0.9,
    metalness: 0.05
  });

  const wireGeometry = new THREE.WireframeGeometry(geometry);
  const wireMaterial = new THREE.LineBasicMaterial({
    color: '#eef4fb',
    transparent: true,
    opacity: 0.35
  });
</script>

<group rotation={[-0.05, 0, 0]}>
  <T.Mesh geometry={geometry} material={material} />
  <T.LineSegments geometry={wireGeometry} material={wireMaterial} />
</group>
