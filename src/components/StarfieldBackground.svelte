<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let container;

    onMount(() => {
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x000000, 0.02);

        const getW = () => container.clientWidth;
        const getH = () => container.clientHeight;

        const camera = new THREE.PerspectiveCamera(75, getW() / getH(), 0.1, 1000);
        camera.position.z = 5;

        const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
        renderer.setSize(getW(), getH());
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        container.appendChild(renderer.domElement);

        // Star field
        const count = 2000;
        const positions = new Float32Array(count * 3);
        for (let i = 0; i < count * 3; i++) positions[i] = (Math.random() - 0.5) * 20;

        const geometry = new THREE.BufferGeometry();
        geometry.setAttribute("position", new THREE.BufferAttribute(positions, 3));

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

        let mouseX = 0;
        let mouseY = 0;

        const handleMouseMove = (e) => {
            mouseX = e.clientX - getW() / 2;
            mouseY = e.clientY - getH() / 2;
        };
        document.addEventListener("mousemove", handleMouseMove);

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

        // Use ResizeObserver on the container so it responds to layout changes too
        const ro = new ResizeObserver(() => {
            const w = getW();
            const h = getH();
            if (w === 0 || h === 0) return;
            camera.aspect = w / h;
            camera.updateProjectionMatrix();
            renderer.setSize(w, h);
        });
        ro.observe(container);

        return () => {
            cancelAnimationFrame(raf);
            ro.disconnect();
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
        overflow: hidden;
    }

    .bg :global(canvas) {
        display: block;
        width: 100% !important;
        height: 100% !important;
    }
</style>