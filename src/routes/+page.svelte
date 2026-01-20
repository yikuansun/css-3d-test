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

    const perspective = 1000;

    let keysPressed: { [key: string]: boolean } = {};

    const TRANSLATION_SPEED = 200; // px / sec
    const ROTATION_SPEED = 150; // deg / sec

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

<svelte:window onkeydown={e => keysPressed[e.key] = true} onkeyup={e => keysPressed[e.key] = false}
    on:click={(e) => {
        if (!document.pointerLockElement) {
            document.body.requestPointerLock();
        }
    }}
    on:mousemove={(e) => {
        if (document.pointerLockElement) {
            camera.angleY += (e.movementX / 1000) * ROTATION_SPEED;
            camera.angleX -= (e.movementY / 1000) * ROTATION_SPEED;
        }
    }} />

<div style:width="960px" style:height="540px" style:background-color="deepskyblue"
    style:position="fixed" style:left="50vw" style:top="50vh" style:overflow="hidden"
    style:transform="translate(-50%, -50%) scale(calc(min(100vw / 960px, 100vh / 540px)))"
    style:perspective="{perspective}px">
    <World perspective={perspective} camera={camera}>
        <!-- Ground -->
        <Plane x={480} y={320} z={0} angleX={90} width={1600} height={1600} color="forestgreen" />

        <!-- House -->
        <Plane x={480} y={270} z={-400} width={100} height={100} color="firebrick" />
        <Plane x={480} y={270} z={-500} width={100} height={100} color="firebrick" brightness={70} />
        <Plane x={530} y={270} z={-450} angleY={90} width={100} height={100} color="firebrick" brightness={85} />
        <Plane x={430} y={270} z={-450} angleY={90} width={100} height={100} color="firebrick" brightness={85} />
        <Plane x={439} y={214} angleX={90} angleY={-35} z={-450} width={100} height={100} color="firebrick" brightness={50} />
        <Plane x={521} y={214} angleX={90} angleY={35} z={-450} width={100} height={100} color="firebrick" brightness={50} />
    </World>
</div>

<span>FPS: {displayedFPS}</span>
<p>
    Camera position: {camera.x}, {camera.y}, {camera.z}
</p>

<style>
</style>