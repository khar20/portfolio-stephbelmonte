<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let container;

    onMount(() => {
        // scene setup
        const scene = new THREE.Scene();

        const camera = new THREE.PerspectiveCamera(50, 1, 0.1, 100);
        camera.position.set(0, 0, 5);

        const renderer = new THREE.WebGLRenderer({
            alpha: true,
            antialias: true,
        });

        const size = container.clientWidth;
        renderer.setSize(size, size);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        container.appendChild(renderer.domElement);

        // model (replace with GLB)
        const geometry = new THREE.IcosahedronGeometry(1.5, 0);
        const material = new THREE.MeshStandardMaterial({
            color: 0xffffff,
            roughness: 0.1,
            metalness: 0.1,
            flatShading: true,
        });

        const mesh = new THREE.Mesh(geometry, material);
        scene.add(mesh);

        // lighting
        const ambient = new THREE.AmbientLight(0xffffff, 0.4);
        scene.add(ambient);

        const key = new THREE.DirectionalLight(0xffffff, 2);
        key.position.set(2, 2, 5);
        scene.add(key);

        const fill = new THREE.PointLight(0x8888ff, 0.6);
        fill.position.set(-5, 0, 2);
        scene.add(fill);

        // mouse interaction (subtle tilt)
        let mouseX = 0;
        let mouseY = 0;

        const handleMouseMove = (e) => {
            const rect = container.getBoundingClientRect();
            mouseX = (e.clientX - rect.left - rect.width / 2) * 0.002;
            mouseY = (e.clientY - rect.top - rect.height / 2) * 0.002;
        };

        container.addEventListener("mousemove", handleMouseMove);

        // animation loop
        const clock = new THREE.Clock();
        let raf;

        function animate() {
            const t = clock.getElapsedTime();

            mesh.rotation.y += 0.004;
            mesh.rotation.x += 0.002;

            mesh.rotation.y += mouseX * 0.02;
            mesh.rotation.x += mouseY * 0.02;

            mesh.position.y = Math.sin(t * 0.6) * 0.15;

            renderer.render(scene, camera);
            raf = requestAnimationFrame(animate);
        }

        animate();

        // resize
        const handleResize = () => {
            const size = container.clientWidth;
            renderer.setSize(size, size);
            camera.aspect = 1;
            camera.updateProjectionMatrix();
        };

        window.addEventListener("resize", handleResize);

        // cleanup
        return () => {
            cancelAnimationFrame(raf);
            window.removeEventListener("resize", handleResize);
            container.removeEventListener("mousemove", handleMouseMove);

            geometry.dispose();
            material.dispose();
            renderer.dispose();
        };
    });
</script>

<div bind:this={container} class="model"></div>

<style>
    .model {
        width: 100%;
        max-width: 500px;
        aspect-ratio: 1;
    }
</style>
