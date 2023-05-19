<script>
  import { onMount, afterUpdate, onDestroy } from 'svelte';
  import * as THREE from 'three';
  
  let container;
  let camera, scene, renderer, stars;
  let starVertices = [];
  
  onMount(async () => {
    camera = new THREE.PerspectiveCamera(60, container.clientWidth / container.clientHeight, 1, 1000);
    camera.position.z = 1;
    scene = new THREE.Scene();
    
    let starVertices = [];
  let starColors = []; // add this line

  for(let i = 0; i < 6000; i++) {
    starVertices.push(
      (Math.random() - 0.5) * 600,  // x
      (Math.random() - 0.5) * 600,  // y
      (Math.random() - 0.5) * 600   // z
    );

    let color;
    if (i % 3 === 0) {
      color = new THREE.Color(0xffffff); // White
    } else if (i % 3 === 1) {
      color = new THREE.Color(0x0000ff); // Blue
    } else {
      color = new THREE.Color(0xffff00); // Yellow
    }

    starColors.push(color.r, color.g, color.b);
  }


  let starGeometry = new THREE.BufferGeometry();
    starGeometry.setAttribute('position', new THREE.Float32BufferAttribute(starVertices, 3));
    starGeometry.setAttribute('color', new THREE.Float32BufferAttribute(starColors, 3));

    let sprite = new THREE.TextureLoader().load("https://threejs.org/examples/textures/sprites/disc.png");
    let starMaterial = new THREE.PointsMaterial({
        size: 2,
        map: sprite,
        vertexColors: true // Enable vertex colors
    });

    stars = new THREE.Points(starGeometry, starMaterial);
    scene.add(stars);
  
    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);
  
    animate();
  });

  function animate() {
    // Get the star positions
    let positions = stars.geometry.attributes.position.array;

    // Update each star's position
    for(let i = 0; i < positions.length; i += 3) {
        // Decrease the z-position to move the star towards the camera
        positions[i + 2] -= 1;

        // If the star has moved past the camera, reset it to the far distance
        if(positions[i + 2] < -600) {
            positions[i + 2] = 600;
        }
    }

    // Let Three.js know the star positions need updating
    stars.geometry.attributes.position.needsUpdate = true;

    renderer.render(scene, camera);
    requestAnimationFrame(animate);
}


  function handleResize() {
    if (!renderer) {
      return;
    }
  
    const width = container.clientWidth;
    const height = container.clientHeight;
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
  }

  // Call handleResize on window resize
  afterUpdate(() => {
    window.addEventListener('resize', handleResize);
  });

  // Cleanup event listener on component destroy
  onDestroy(() => {
    window.removeEventListener('resize', handleResize);
  });
</script>

<svelte:window on:resize={handleResize} />

<div bind:this={container} style="width: 100%; height: 100vh"></div>
