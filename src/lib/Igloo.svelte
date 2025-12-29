<script>
  import { onMount } from 'svelte';
  import { T, useFrame } from '@threlte/core';
  import { gsap } from 'gsap';
  import * as THREE from 'three';

  export let heroMix = 1;
  export let interiorMix = 0;

  let wireOpacity = 1;
  let solidOpacity = 0;

  const baseGeometry = new THREE.SphereGeometry(2.6, 16, 10, 0, Math.PI * 2, 0, Math.PI * 0.9);
  baseGeometry.scale(1, 0.85, 1);

  const wireGeometry = new THREE.WireframeGeometry(baseGeometry);
  wireGeometry.computeLineDistances();

  const uniforms = {
    uTime: { value: 0 },
    uDashSize: { value: 0.32 },
    uGapSize: { value: 0.22 },
    uGlow: { value: 1.5 },
    uOpacity: { value: 1 }
  };

  const wireMaterial = new THREE.ShaderMaterial({
    uniforms,
    transparent: true,
    vertexShader: `
      attribute float lineDistance;
      varying float vLineDistance;
      void main() {
        vLineDistance = lineDistance;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      uniform float uTime;
      uniform float uDashSize;
      uniform float uGapSize;
      uniform float uGlow;
      uniform float uOpacity;
      varying float vLineDistance;

      void main() {
        float totalSize = uDashSize + uGapSize;
        float phase = mod(vLineDistance * 0.5 - uTime, totalSize);
        float dash = smoothstep(uDashSize, uDashSize - 0.05, phase);
        float glow = smoothstep(0.0, uDashSize, phase) * uGlow;
        vec3 color = vec3(0.87, 0.93, 1.0) * glow;
        float alpha = dash * uOpacity;
        gl_FragColor = vec4(color, alpha);
      }
    `
  });

  const solidMaterial = new THREE.MeshPhysicalMaterial({
    color: '#dfe9f2',
    roughness: 0.35,
    transmission: 0.55,
    thickness: 1.2,
    ior: 1.2,
    metalness: 0.0,
    clearcoat: 0.4,
    transparent: true,
    opacity: 0
  });

  let revealTimeline;

  onMount(() => {
    revealTimeline = gsap.timeline();
    revealTimeline
      .to(uniforms.uTime, { value: 4, duration: 2.6, ease: 'power2.inOut' })
      .to(
        {},
        {
          duration: 0.1,
          onComplete: () => {
            gsap.to({ value: wireOpacity }, {
              value: 0,
              duration: 1,
              ease: 'power2.out',
              onUpdate() {
                wireOpacity = this.targets()[0].value;
                uniforms.uOpacity.value = wireOpacity;
              }
            });
            gsap.to({ value: solidOpacity }, {
              value: 1,
              duration: 1.2,
              ease: 'power2.out',
              onUpdate() {
                solidOpacity = this.targets()[0].value;
                solidMaterial.opacity = solidOpacity;
              }
            });
          }
        }
      );

    return () => {
      revealTimeline?.kill();
    };
  });

  useFrame((_, delta) => {
    uniforms.uTime.value += delta * 0.6;
    uniforms.uOpacity.value = wireOpacity * heroMix;
    solidMaterial.opacity = solidOpacity * (0.2 + interiorMix * 0.8);
  });
</script>

<group position={[0, 0.6, 0]}>
  <T.LineSegments geometry={wireGeometry} material={wireMaterial} />
  <T.Mesh geometry={baseGeometry} material={solidMaterial} />
</group>
