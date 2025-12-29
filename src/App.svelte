<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

  let container;

  onMount(() => {
    // 1. Create the Scene
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x1a1a2e);

    // 2. Create the Camera
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    camera.position.z = 5;

    // 3. Create the Renderer
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    container.appendChild(renderer.domElement);

    // 4. Add Orbit Controls
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.enableZoom = true;
    controls.enablePan = true;

    // 5. Create a Cube
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshStandardMaterial({ 
      color: 0x6366f1,
      metalness: 0.3,
      roughness: 0.4
    });
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    // 6. Add a second object - Icosahedron (ice-like shape)
    const icoGeometry = new THREE.IcosahedronGeometry(0.7, 0);
    const icoMaterial = new THREE.MeshStandardMaterial({
      color: 0x88ccff,
      metalness: 0.2,
      roughness: 0.1,
      transparent: true,
      opacity: 0.8
    });
    const icosahedron = new THREE.Mesh(icoGeometry, icoMaterial);
    icosahedron.position.x = 2.5;
    scene.add(icosahedron);

    // 7. Add a Torus Knot
    const torusGeometry = new THREE.TorusKnotGeometry(0.5, 0.15, 100, 16);
    const torusMaterial = new THREE.MeshStandardMaterial({
      color: 0xff6b9d,
      metalness: 0.5,
      roughness: 0.2
    });
    const torusKnot = new THREE.Mesh(torusGeometry, torusMaterial);
    torusKnot.position.x = -2.5;
    scene.add(torusKnot);

    // 8. Add Lighting
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    const pointLight = new THREE.PointLight(0xffffff, 1);
    pointLight.position.set(5, 5, 5);
    scene.add(pointLight);

    const pointLight2 = new THREE.PointLight(0x88ccff, 0.5);
    pointLight2.position.set(-5, -5, 5);
    scene.add(pointLight2);

    // 9. Animation Loop
    function animate() {
      requestAnimationFrame(animate);
      
      // Rotate objects
      cube.rotation.x += 0.005;
      cube.rotation.y += 0.005;
      
      icosahedron.rotation.x += 0.008;
      icosahedron.rotation.y += 0.006;
      
      torusKnot.rotation.x += 0.007;
      torusKnot.rotation.y += 0.009;

      // Update controls
      controls.update();

      renderer.render(scene, camera);
    }
    animate();

    // 10. Handle Window Resize
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  });
</script>

<div bind:this={container} class="canvas-container"></div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
  }

  .canvas-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
</style>