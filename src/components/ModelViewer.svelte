<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let container;

    onMount(() => {
        const scene = new THREE.Scene();

        // container's actual aspect
        const getAspect = () => container.clientWidth / container.clientHeight;
        const camera = new THREE.PerspectiveCamera(45, getAspect(), 0.1, 100);
        camera.position.set(0, 0, 5);

        const renderer = new THREE.WebGLRenderer({
            alpha: true,
            antialias: true,
        });
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.setSize(container.clientWidth, container.clientHeight);
        container.appendChild(renderer.domElement);

        const geometry = new THREE.IcosahedronGeometry(1.5, 0);
        const material = new THREE.MeshStandardMaterial({
            color: 0xffffff,
            roughness: 0.15,
            metalness: 0.05,
            flatShading: true,
        });
        const mesh = new THREE.Mesh(geometry, material);
        scene.add(mesh);

        // lighting
        scene.add(new THREE.AmbientLight(0xffffff, 0.4));

        const key = new THREE.DirectionalLight(0xffffff, 2);
        key.position.set(2, 2, 5);
        scene.add(key);

        const fill = new THREE.PointLight(0x8888ff, 0.6);
        fill.position.set(-5, 0, 2);
        scene.add(fill);

        // mouse tilt
        let mouseX = 0;
        let mouseY = 0;

        const handleMouseMove = (e) => {
            const rect = container.getBoundingClientRect();
            mouseX = ((e.clientX - rect.left) / rect.width - 0.5) * 2;
            mouseY = ((e.clientY - rect.top) / rect.height - 0.5) * 2;
        };

        // reset on leave
        const handleMouseLeave = () => {
            mouseX = 0;
            mouseY = 0;
        };

        container.addEventListener("mousemove", handleMouseMove);
        container.addEventListener("mouseleave", handleMouseLeave);

        // resize reacts to container size changes
        const ro = new ResizeObserver(() => {
            const w = container.clientWidth;
            const h = container.clientHeight;
            if (w === 0 || h === 0) return;
            renderer.setSize(w, h);
            camera.aspect = w / h;
            camera.updateProjectionMatrix();
        });
        ro.observe(container);

        // animation
        const clock = new THREE.Clock();
        let raf;

        function animate() {
            const t = clock.getElapsedTime();
            mesh.rotation.y += 0.004 + mouseX * 0.015;
            mesh.rotation.x += 0.002 + mouseY * 0.008;
            mesh.position.y = Math.sin(t * 0.6) * 0.15;
            renderer.render(scene, camera);
            raf = requestAnimationFrame(animate);
        }

        animate();

        return () => {
            cancelAnimationFrame(raf);
            ro.disconnect();
            container.removeEventListener("mousemove", handleMouseMove);
            container.removeEventListener("mouseleave", handleMouseLeave);
            geometry.dispose();
            material.dispose();
            renderer.dispose();
        };
    });
</script>

<div bind:this={container} class="model-canvas"></div>

<style>
    .model-canvas {
        width: 100%;
        height: 100%;
        display: block;
    }

    .model-canvas :global(canvas) {
        display: block;
        width: 100% !important;
        height: 100% !important;
    }
</style>
