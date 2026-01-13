<script lang="ts">
    import Plane from "$lib/components/Plane.svelte";
    import World from "$lib/components/World.svelte";
	import { onDestroy, onMount } from "svelte";

    let camera = $state({
        x: 0,
        y: 0,
        z: 0,
        angleX: 0,
        angleY: 0,
        angleZ: 0,
    });

    const perspective = 300;

    let keysPressed: { [key: string]: boolean } = {};

    const TRANSLATION_SPEED = 100; // px / sec
    const ROTATION_SPEED = 100; // deg / sec

    let lastTime: number;
    let deltaTime = $state(0);
    let displayedFPS: number = $state(0);
    let animationFrameId: number;
    let frameCount = 0;
    function tick() {
        let startTime = Date.now();
        deltaTime = (startTime - lastTime) / 1000;
        lastTime = startTime;
        // REMEMBER: multiply all speeds by deltaTime

        if (keysPressed["w"]) {
            camera.z -= TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180) * deltaTime;
            camera.x += TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180) * deltaTime;
        }
        if (keysPressed["s"]) {
            camera.z += TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180) * deltaTime;
            camera.x -= TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180) * deltaTime;
        }
        if (keysPressed["a"]) {
            camera.x -= TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180) * deltaTime;
            camera.z -= TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180) * deltaTime;
        }
        if (keysPressed["d"]) {
            camera.x += TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180) * deltaTime;
            camera.z += TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180) * deltaTime;
        }

        // camera rotation
        if (keysPressed["ArrowLeft"]) camera.angleY -= ROTATION_SPEED * deltaTime;
        if (keysPressed["ArrowRight"]) camera.angleY += ROTATION_SPEED * deltaTime;
        if (keysPressed["ArrowUp"]) camera.angleX += ROTATION_SPEED * deltaTime;
        if (keysPressed["ArrowDown"]) camera.angleX -= ROTATION_SPEED * deltaTime;

        if (frameCount % 60 === 0) displayedFPS = Math.floor(1 / deltaTime);
        frameCount++;

        animationFrameId = requestAnimationFrame(tick);
    }

    onMount(() => {
        lastTime = Date.now();
        animationFrameId = requestAnimationFrame(tick);
    });

    onDestroy(() => {
        if (animationFrameId) cancelAnimationFrame(animationFrameId);
    });
</script>

<svelte:window onkeydown={e => keysPressed[e.key] = true} onkeyup={e => keysPressed[e.key] = false} />

<div style:width="960px" style:height="540px" style:background-color="black"
    style:position="fixed" style:left="50vw" style:top="50vh" style:overflow="hidden"
    style:transform="translate(-50%, -50%) scale(calc(min(100vw / 960px, 100vh / 540px)))"
    style:perspective="{perspective}px">
    <World perspective={perspective} camera={camera}>
        <Plane x={480} y={270} z={-50} width={100} height={100} color="red" />
        <Plane x={480} y={270} z={50} width={100} height={100} color="blue" />
        <Plane x={430} y={270} z={0} angleY={90} width={100} height={100} color="green" />
        <Plane x={530} y={270} z={0} angleY={90} width={100} height={100} color="yellow" />
    </World>
</div>

<span>FPS: {displayedFPS}</span>

<style>
</style>