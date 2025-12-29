<script>
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  import gsap from 'gsap';
  import { ScrollTrigger } from 'gsap/ScrollTrigger';

  gsap.registerPlugin(ScrollTrigger);

  let container;
  let introComplete = false;
  let loadingProgress = 0;
  let currentSection = 0;

  const projects = [
    { name: 'Pudgy Penguins', description: 'The beloved NFT collection', color: 0x88ccff },
    { name: 'Lil Pudgys', description: 'The next generation', color: 0xffb6c1 },
    { name: 'Pudgy Rods', description: 'Fishing for the future', color: 0x98fb98 }
  ];

  onMount(() => {
    // Scene setup
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0a0a12);
    scene.fog = new THREE.Fog(0x0a0a12, 5, 25);

    const camera = new THREE.PerspectiveCamera(
      60,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    camera.position.set(0, 0, 8);

    const renderer = new THREE.WebGLRenderer({ 
      antialias: true,
      alpha: true,
      powerPreference: 'high-performance'
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.toneMappingExposure = 1.2;
    container.appendChild(renderer.domElement);

    // Lighting
    const ambientLight = new THREE.AmbientLight(0x404060, 0.5);
    scene.add(ambientLight);

    const mainLight = new THREE.DirectionalLight(0xffffff, 1);
    mainLight.position.set(5, 10, 7);
    scene.add(mainLight);

    const rimLight = new THREE.DirectionalLight(0x88ccff, 0.5);
    rimLight.position.set(-5, 5, -5);
    scene.add(rimLight);

    const pointLight1 = new THREE.PointLight(0x6366f1, 1, 15);
    pointLight1.position.set(3, 2, 4);
    scene.add(pointLight1);

    const pointLight2 = new THREE.PointLight(0x22d3ee, 0.8, 15);
    pointLight2.position.set(-3, -2, 4);
    scene.add(pointLight2);

    // Create ice material
    const createIceMaterial = (color) => {
      return new THREE.MeshPhysicalMaterial({
        color: color,
        metalness: 0.0,
        roughness: 0.1,
        transmission: 0.9,
        thickness: 1.5,
        envMapIntensity: 1,
        clearcoat: 1,
        clearcoatRoughness: 0.1,
        ior: 1.5,
        transparent: true,
        opacity: 0.9
      });
    };

    // Create ice block geometry
    const createIceBlock = (index) => {
      const group = new THREE.Group();
      
      // Main ice crystal
      const icosaGeo = new THREE.IcosahedronGeometry(1.2, 1);
      const iceMat = createIceMaterial(projects[index].color);
      const iceBlock = new THREE.Mesh(icosaGeo, iceMat);
      
      // Add random vertices displacement
      const positions = icosaGeo.attributes.position;
      for (let i = 0; i < positions.count; i++) {
        const x = positions.getX(i);
        const y = positions.getY(i);
        const z = positions.getZ(i);
        const noise = (Math.random() - 0.5) * 0.2;
        positions.setXYZ(i, x + noise, y + noise, z + noise);
      }
      icosaGeo.computeVertexNormals();
      
      group.add(iceBlock);

      // Inner object
      const innerGeo = new THREE.OctahedronGeometry(0.5, 0);
      const innerMat = new THREE.MeshStandardMaterial({
        color: 0xffffff,
        emissive: projects[index].color,
        emissiveIntensity: 0.3,
        metalness: 0.8,
        roughness: 0.2
      });
      const innerObject = new THREE.Mesh(innerGeo, innerMat);
      group.add(innerObject);

      // Small ice crystals around
      for (let i = 0; i < 8; i++) {
        const crystalGeo = new THREE.ConeGeometry(0.1, 0.4, 4);
        const crystalMat = createIceMaterial(0xffffff);
        const crystal = new THREE.Mesh(crystalGeo, crystalMat);
        
        const angle = (i / 8) * Math.PI * 2;
        const radius = 1.5 + Math.random() * 0.5;
        crystal.position.set(
          Math.cos(angle) * radius,
          (Math.random() - 0.5) * 1.5,
          Math.sin(angle) * radius
        );
        crystal.rotation.set(
          Math.random() * Math.PI,
          Math.random() * Math.PI,
          Math.random() * Math.PI
        );
        crystal.scale.setScalar(0.5 + Math.random() * 0.5);
        group.add(crystal);
      }

      return { group, iceBlock, innerObject };
    };

    // Create ice blocks for each project
    const iceBlocks = projects.map((_, index) => {
      const block = createIceBlock(index);
      block.group.position.x = index * 6 - 6;
      block.group.visible = false;
      scene.add(block.group);
      return block;
    });

    // Particles system
    const particlesGeometry = new THREE.BufferGeometry();
    const particlesCount = 2000;
    const posArray = new Float32Array(particlesCount * 3);
    
    for (let i = 0; i < particlesCount * 3; i++) {
      posArray[i] = (Math.random() - 0.5) * 30;
    }
    
    particlesGeometry.setAttribute('position', new THREE.BufferAttribute(posArray, 3));
    
    const particlesMaterial = new THREE.PointsMaterial({
      size: 0.02,
      color: 0xb6bac5,
      transparent: true,
      opacity: 0.6,
      blending: THREE.AdditiveBlending
    });
    
    const particlesMesh = new THREE.Points(particlesGeometry, particlesMaterial);
    scene.add(particlesMesh);

    // Intro animation
    const introTimeline = gsap.timeline({
      onComplete: () => {
        introComplete = true;
        iceBlocks.forEach(block => block.group.visible = true);
      }
    });

    // Loading simulation
    gsap.to({}, {
      duration: 2,
      onUpdate: function() {
        loadingProgress = Math.round(this.progress() * 100);
      }
    });

    introTimeline
      .to(camera.position, {
        z: 5,
        duration: 2,
        ease: 'power2.inOut',
        delay: 2.5
      })
      .to(particlesMesh.rotation, {
        y: Math.PI * 0.5,
        duration: 2,
        ease: 'power2.inOut'
      }, '<');

    // Scroll-based navigation
    let scrollY = 0;
    let targetScrollY = 0;

    const handleScroll = () => {
      if (!introComplete) return;
      
      targetScrollY = window.scrollY;
      const sectionHeight = window.innerHeight;
      currentSection = Math.round(targetScrollY / sectionHeight);
      currentSection = Math.max(0, Math.min(currentSection, projects.length - 1));
    };

    window.addEventListener('scroll', handleScroll);

    // Mouse movement for parallax
    let mouseX = 0;
    let mouseY = 0;

    const handleMouseMove = (event) => {
      mouseX = (event.clientX / window.innerWidth) * 2 - 1;
      mouseY = -(event.clientY / window.innerHeight) * 2 + 1;
    };

    window.addEventListener('mousemove', handleMouseMove);

    // Animation loop
    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);
      
      const elapsedTime = clock.getElapsedTime();
      
      scrollY += (targetScrollY - scrollY) * 0.1;
      
      particlesMesh.rotation.y = elapsedTime * 0.05;
      particlesMesh.rotation.x = Math.sin(elapsedTime * 0.1) * 0.1;

      iceBlocks.forEach((block, index) => {
        if (block.group.visible) {
          block.group.position.y = Math.sin(elapsedTime + index) * 0.3;
          
          block.iceBlock.rotation.y = elapsedTime * 0.3 + index;
          block.iceBlock.rotation.x = Math.sin(elapsedTime * 0.5) * 0.2;
          
          block.innerObject.rotation.y = -elapsedTime * 0.5;
          block.innerObject.rotation.x = elapsedTime * 0.3;
          
          const targetScale = currentSection === index ? 1 : 0.7;
          block.group.scale.lerp(new THREE.Vector3(targetScale, targetScale, targetScale), 0.05);
        }
      });

      if (introComplete) {
        const targetX = currentSection * 6 - 6;
        camera.position.x += (targetX - camera.position.x) * 0.05;
        
        camera.position.x += mouseX * 0.3;
        camera.position.y += mouseY * 0.2;
        camera.lookAt(targetX, 0, 0);
      }

      pointLight1.position.x = Math.sin(elapsedTime) * 3;
      pointLight2.position.x = Math.cos(elapsedTime) * 3;

      renderer.render(scene, camera);
    }
    animate();

    const handleResize = () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    };

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('scroll', handleScroll);
      window.removeEventListener('mousemove', handleMouseMove);
      window.removeEventListener('resize', handleResize);
    };
  });
</script>

{#if !introComplete}
<div class="intro-screen">
  <div class="intro-content">
    <div class="logo-glitch" data-text="IGLOO">IGLOO</div>
    <div class="loading-bar">
      <div class="loading-progress" style="width: {loadingProgress}%"></div>
    </div>
    <div class="loading-text">{loadingProgress}%</div>
  </div>
  <div class="scan-line"></div>
</div>
{/if}

<div bind:this={container} class="canvas-container"></div>

<div class="scroll-container">
  {#each projects as project, index}
    <section class="project-section" class:active={currentSection === index}>
      <div class="project-info">
        <span class="project-number">0{index + 1}</span>
        <h2 class="project-title">{project.name}</h2>
        <p class="project-description">{project.description}</p>
        <button class="explore-btn">
          <span>Explore</span>
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
            <path d="M5 12H19M19 12L12 5M19 12L12 19" stroke="currentColor" stroke-width="2"/>
          </svg>
        </button>
      </div>
    </section>
  {/each}
</div>

<nav class="main-nav">
  <div class="nav-logo">IGLOO INC.</div>
  <div class="nav-links">
    <a href="#about">About</a>
    <a href="#projects">Projects</a>
    <a href="#contact">Contact</a>
  </div>
</nav>

<div class="section-indicators">
  {#each projects as _, index}
    <button 
      class="indicator" 
      class:active={currentSection === index}
      on:click={() => window.scrollTo({ top: index * window.innerHeight, behavior: 'smooth' })}
    ></button>
  {/each}
</div>

<footer class="main-footer">
  <div class="footer-content">
    <p>Creating the largest onchain community</p>
    <div class="social-links">
      <a href="#twitter">Twitter</a>
      <a href="#discord">Discord</a>
    </div>
  </div>
</footer>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');
  
  :global(*) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  :global(body) {
    font-family: 'Space Grotesk', sans-serif;
    background: #0a0a12;
    color: #b6bac5;
    overflow-x: hidden;
  }

  :global(html) {
    scroll-behavior: smooth;
  }

  .canvas-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
  }

  .intro-screen {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: #0a0a12;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    overflow: hidden;
  }

  .intro-content {
    text-align: center;
    z-index: 2;
  }

  .logo-glitch {
    font-family: 'JetBrains Mono', monospace;
    font-size: clamp(3rem, 10vw, 8rem);
    font-weight: 700;
    color: #b6bac5;
    letter-spacing: 0.3em;
    position: relative;
    animation: glitch 2s infinite;
  }

  .logo-glitch::before,
  .logo-glitch::after {
    content: attr(data-text);
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }

  .logo-glitch::before {
    color: #ff006e;
    animation: glitch-1 0.3s infinite;
    clip-path: polygon(0 0, 100% 0, 100% 35%, 0 35%);
  }

  .logo-glitch::after {
    color: #00f5ff;
    animation: glitch-2 0.3s infinite;
    clip-path: polygon(0 65%, 100% 65%, 100% 100%, 0 100%);
  }

  @keyframes glitch {
    0%, 90%, 100% { transform: translate(0); }
    92% { transform: translate(-2px, 2px); }
    94% { transform: translate(2px, -2px); }
    96% { transform: translate(-2px, -2px); }
    98% { transform: translate(2px, 2px); }
  }

  @keyframes glitch-1 {
    0%, 100% { transform: translate(0); }
    20% { transform: translate(-3px, 3px); }
    40% { transform: translate(3px, -3px); }
    60% { transform: translate(-3px, -3px); }
    80% { transform: translate(3px, 3px); }
  }

  @keyframes glitch-2 {
    0%, 100% { transform: translate(0); }
    20% { transform: translate(3px, -3px); }
    40% { transform: translate(-3px, 3px); }
    60% { transform: translate(3px, 3px); }
    80% { transform: translate(-3px, -3px); }
  }

  .loading-bar {
    width: 200px;
    height: 2px;
    background: rgba(182, 186, 197, 0.2);
    margin: 2rem auto;
    overflow: hidden;
  }

  .loading-progress {
    height: 100%;
    background: linear-gradient(90deg, #6366f1, #22d3ee);
    transition: width 0.1s ease;
  }

  .loading-text {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.875rem;
    color: rgba(182, 186, 197, 0.6);
  }

  .scan-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(90deg, transparent, rgba(99, 102, 241, 0.5), transparent);
    animation: scan 2s linear infinite;
  }

  @keyframes scan {
    0% { top: 0; }
    100% { top: 100%; }
  }

  .scroll-container {
    position: relative;
    z-index: 10;
    pointer-events: none;
  }

  .project-section {
    height: 100vh;
    display: flex;
    align-items: center;
    padding: 0 5vw;
    opacity: 0.3;
    transition: opacity 0.5s ease;
    pointer-events: auto;
  }

  .project-section.active {
    opacity: 1;
  }

  .project-info {
    max-width: 500px;
  }

  .project-number {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.875rem;
    color: #6366f1;
    display: block;
    margin-bottom: 1rem;
  }

  .project-title {
    font-size: clamp(2rem, 5vw, 4rem);
    font-weight: 600;
    color: #ffffff;
    margin-bottom: 1rem;
    line-height: 1.1;
  }

  .project-description {
    font-size: 1.125rem;
    color: rgba(182, 186, 197, 0.8);
    margin-bottom: 2rem;
    line-height: 1.6;
  }

  .explore-btn {
    display: inline-flex;
    align-items: center;
    gap: 0.75rem;
    padding: 1rem 2rem;
    background: transparent;
    border: 1px solid rgba(182, 186, 197, 0.3);
    color: #b6bac5;
    font-family: 'Space Grotesk', sans-serif;
    font-size: 0.875rem;
    font-weight: 500;
    letter-spacing: 0.1em;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .explore-btn:hover {
    background: rgba(99, 102, 241, 0.1);
    border-color: #6366f1;
    color: #ffffff;
  }

  .explore-btn svg {
    transition: transform 0.3s ease;
  }

  .explore-btn:hover svg {
    transform: translateX(5px);
  }

  .main-nav {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 2rem 5vw;
    z-index: 50;
  }

  .nav-logo {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.875rem;
    font-weight: 600;
    letter-spacing: 0.2em;
    color: #ffffff;
  }

  .nav-links {
    display: flex;
    gap: 3rem;
  }

  .nav-links a {
    font-size: 0.875rem;
    color: rgba(182, 186, 197, 0.8);
    text-decoration: none;
    transition: color 0.3s ease;
  }

  .nav-links a:hover {
    color: #ffffff;
  }

  .section-indicators {
    position: fixed;
    right: 2rem;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    flex-direction: column;
    gap: 1rem;
    z-index: 50;
  }

  .indicator {
    width: 12px;
    height: 12px;
    border-radius: 50%;
    border: 1px solid rgba(182, 186, 197, 0.4);
    background: transparent;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  .indicator:hover {
    border-color: #6366f1;
  }

  .indicator.active {
    background: #6366f1;
    border-color: #6366f1;
    box-shadow: 0 0 20px rgba(99, 102, 241, 0.5);
  }

  .main-footer {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    padding: 2rem 5vw;
    display: flex;
    justify-content: space-between;
    align-items: center;
    z-index: 50;
    pointer-events: none;
  }

  .footer-content {
    display: flex;
    justify-content: space-between;
    width: 100%;
    align-items: center;
    pointer-events: auto;
  }

  .footer-content p {
    font-size: 0.875rem;
    color: rgba(182, 186, 197, 0.5);
  }

  .social-links {
    display: flex;
    gap: 2rem;
  }

  .social-links a {
    font-size: 0.875rem;
    color: rgba(182, 186, 197, 0.5);
    text-decoration: none;
    transition: color 0.3s ease;
  }

  .social-links a:hover {
    color: #6366f1;
  }

  @media (max-width: 768px) {
    .nav-links {
      display: none;
    }

    .section-indicators {
      right: 1rem;
    }

    .project-info {
      max-width: 100%;
    }
  }
</style>