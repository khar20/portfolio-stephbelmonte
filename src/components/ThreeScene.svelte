<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let canvasContainer;

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
        canvasContainer.appendChild(renderer.domElement);

        // star field (background system)
        const starGeometry = new THREE.BufferGeometry();
        const starCount = 2000;
        const starPositions = new Float32Array(starCount * 3);

        for (let i = 0; i < starCount * 3; i++) {
            starPositions[i] = (Math.random() - 0.5) * 20;
        }

        starGeometry.setAttribute(
            "position",
            new THREE.BufferAttribute(starPositions, 3),
        );

        const starMaterial = new THREE.PointsMaterial({
            size: 0.02,
            color: 0xffffff,
            transparent: true,
            opacity: 0.8,
            blending: THREE.AdditiveBlending,
            depthWrite: false, // stars stay in background
        });

        const starfield = new THREE.Points(starGeometry, starMaterial);
        starfield.renderOrder = 0;
        scene.add(starfield);

        // main model (foreground system)
        // todo: replace placeholder geometry with actual model
        const geoGeometry = new THREE.IcosahedronGeometry(1.5, 0);
        const geoMaterial = new THREE.MeshStandardMaterial({
            color: 0xffffff,
            roughness: 0.1,
            metalness: 0.1,
            flatShading: true,
        });

        const model = new THREE.Mesh(geoGeometry, geoMaterial);
        model.position.set(3, 0, 0);
        model.renderOrder = 1;
        scene.add(model);

        // lighting
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
        scene.add(ambientLight);

        const mainLight = new THREE.DirectionalLight(0xffffff, 2);
        mainLight.position.set(2, 2, 5);
        scene.add(mainLight);

        const fillLight = new THREE.PointLight(0x8888ff, 0.5);
        fillLight.position.set(-5, 0, 2);
        scene.add(fillLight);

        // mouse interaction
        let mouseX = 0;
        let mouseY = 0;

        const handleMouseMove = (event) => {
            mouseX = event.clientX - window.innerWidth / 2;
            mouseY = event.clientY - window.innerHeight / 2;
        };

        document.addEventListener("mousemove", handleMouseMove);

        // animation loop
        const clock = new THREE.Clock();
        let animationId;

        function animate() {
            const elapsedTime = clock.getElapsedTime();

            // main model animation
            model.rotation.y += 0.005;
            model.rotation.x += 0.002;
            model.position.y = Math.sin(elapsedTime * 0.5) * 0.2;

            // starfield animation (parallax)
            starfield.rotation.y = elapsedTime * 0.05;
            starfield.rotation.x = mouseY * 0.00005;

            starfield.position.x +=
                (mouseX * 0.0005 - starfield.position.x) * 0.05;
            starfield.position.y +=
                (-mouseY * 0.0005 - starfield.position.y) * 0.05;

            renderer.render(scene, camera);
            animationId = requestAnimationFrame(animate);
        }

        animate();

        const handleResize = () => {
            const isMobile = window.innerWidth < 768;

            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);

            if (isMobile) {
                model.position.set(0, 1.5, 0);
                model.scale.set(0.8, 0.8, 0.8);
            } else {
                model.position.set(3, 0, 0);
                model.scale.set(1, 1, 1);
            }
        };

        window.addEventListener("resize", handleResize);
        handleResize();

        // cleanup
        return () => {
            cancelAnimationFrame(animationId);

            window.removeEventListener("resize", handleResize);
            document.removeEventListener("mousemove", handleMouseMove);

            // dispose geometries & materials
            starGeometry.dispose();
            starMaterial.dispose();

            geoGeometry.dispose();
            geoMaterial.dispose();

            renderer.dispose();

            // remove canvas
            if (
                renderer.domElement &&
                canvasContainer.contains(renderer.domElement)
            ) {
                canvasContainer.removeChild(renderer.domElement);
            }
        };
    });
</script>

<div bind:this={canvasContainer} class="webgl-container"></div>

<style>
    .webgl-container {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 1;
    }
</style>
