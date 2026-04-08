<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let container;

    onMount(() => {
        // scene setup
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x000000, 0.02);

        const camera = new THREE.PerspectiveCamera(
            75,
            window.innerWidth / window.innerHeight,
            0.1,
            1000,
        );
        camera.position.z = 5;

        const renderer = new THREE.WebGLRenderer({
            alpha: true,
            antialias: true,
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        container.appendChild(renderer.domElement);

        // star field
        const geometry = new THREE.BufferGeometry();
        const count = 2000;
        const positions = new Float32Array(count * 3);

        for (let i = 0; i < count * 3; i++) {
            positions[i] = (Math.random() - 0.5) * 20;
        }

        geometry.setAttribute(
            "position",
            new THREE.BufferAttribute(positions, 3),
        );

        const material = new THREE.PointsMaterial({
            size: 0.02,
            color: 0xffffff,
            transparent: true,
            opacity: 0.8,
            blending: THREE.AdditiveBlending,
            depthWrite: false,
        });

        const stars = new THREE.Points(geometry, material);
        scene.add(stars);

        // mouse interaction
        let mouseX = 0;
        let mouseY = 0;

        const handleMouseMove = (e) => {
            mouseX = e.clientX - window.innerWidth / 2;
            mouseY = e.clientY - window.innerHeight / 2;
        };

        document.addEventListener("mousemove", handleMouseMove);

        // animation loop
        const clock = new THREE.Clock();
        let raf;

        function animate() {
            const t = clock.getElapsedTime();

            stars.rotation.y = t * 0.05;
            stars.rotation.x = mouseY * 0.00005;

            stars.position.x += (mouseX * 0.0005 - stars.position.x) * 0.05;
            stars.position.y += (-mouseY * 0.0005 - stars.position.y) * 0.05;

            renderer.render(scene, camera);
            raf = requestAnimationFrame(animate);
        }

        animate();

        // resize
        const handleResize = () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        };

        window.addEventListener("resize", handleResize);

        // cleanup
        return () => {
            cancelAnimationFrame(raf);
            window.removeEventListener("resize", handleResize);
            document.removeEventListener("mousemove", handleMouseMove);

            geometry.dispose();
            material.dispose();
            renderer.dispose();
        };
    });
</script>

<div bind:this={container} class="bg"></div>

<style>
    .bg {
        position: absolute;
        inset: 0;
        z-index: 1;
        pointer-events: none;
    }
</style>
