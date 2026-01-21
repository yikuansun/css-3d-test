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
        xOffset: 480,
        yOffset: 270,
    });

    const TEXTURE_NAMES: { [key: number]: string } = {
        1: "grass",
        2: "dirt",
    };

    const CUBE_SIZE = 48;

    const perspective = 1024;

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

    let worldMatrix: Array<Array<Array<number>>> = $state([]);

    function createWorldMatrix(size: number) {
        worldMatrix = [];
        for (let x = 0; x < size; x++) {
            worldMatrix.push([]);
            for (let y = 0; y < size; y++) {
                worldMatrix[x].push([]);
                for (let z = 0; z < size; z++) {
                    worldMatrix[x][y].push(0);
                    if (y > size - 3) worldMatrix[x][y][z] = 1;
                }
            }
        }
        worldMatrix[0][7][4] = 1;
        worldMatrix[2][7][2] = 1

        console.log(worldMatrix);
    }

    let pointerEnabled = $state(false);

    onMount(() => {
        createWorldMatrix(10);
        lastTime = Date.now();
        animationFrameId = requestAnimationFrame(tick);
        pointerEnabled = document?.pointerLockElement !== null;
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

<svelte:document onpointerlockchange={() => { pointerEnabled = document?.pointerLockElement !== null; }} />

<div style:width="960px" style:height="540px" style:background-color="deepskyblue"
    style:position="fixed" style:left="50vw" style:top="50vh" style:overflow="hidden"
    style:transform="translate(-50%, -50%) scale(calc(min(100vw / 960px, 100vh / 540px)))"
    style:perspective="{perspective}px">
    <World perspective={perspective} camera={camera}>
        {#each worldMatrix as row, layerX}
            {#each row as column, layerY}
                {#each column as cell, layerZ}
                    {@const cubeX = (layerX - worldMatrix.length / 2) * CUBE_SIZE}
                    {@const cubeY = (layerY - worldMatrix.length / 2) * CUBE_SIZE}
                    {@const cubeZ = (layerZ - worldMatrix.length / 2) * CUBE_SIZE}
                    {#if cell > 0}
                        {#if worldMatrix?.[layerX]?.[layerY - 1]?.[layerZ] === 0}
                            <!-- Top -->
                            <Plane x={cubeX} y={cubeY - CUBE_SIZE / 2} z={cubeZ} angleX={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_top.png" brightness={120} />
                        {/if}
                        {#if worldMatrix?.[layerX]?.[layerY + 1]?.[layerZ] === 0}
                            <!-- Bottom -->
                            <Plane x={cubeX} y={cubeY + CUBE_SIZE / 2} z={cubeZ} angleX={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_bottom.png" brightness={40} />
                        {/if}
                        {#if worldMatrix?.[layerX - 1]?.[layerY]?.[layerZ] === 0}
                            <!-- Left -->
                            <Plane x={cubeX - CUBE_SIZE / 2} y={cubeY} z={cubeZ} angleY={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={60} />
                        {/if}
                        {#if worldMatrix?.[layerX + 1]?.[layerY]?.[layerZ] === 0}
                            <!-- Right -->
                            <Plane x={cubeX + CUBE_SIZE / 2} y={cubeY} z={cubeZ} angleY={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={100} />
                        {/if}
                        {#if worldMatrix?.[layerX]?.[layerY]?.[layerZ - 1] === 0}
                            <!-- Front -->
                            <Plane x={cubeX} y={cubeY} z={cubeZ - CUBE_SIZE / 2} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={80} />
                        {/if}
                        {#if worldMatrix?.[layerX]?.[layerY]?.[layerZ + 1] === 0}
                            <!-- Back -->
                            <Plane x={cubeX} y={cubeY} z={cubeZ + CUBE_SIZE / 2} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={80} />
                        {/if}
                    {/if}
                {/each}
            {/each}
        {/each}
    </World>
    {#if pointerEnabled}
        <div style:pointer-events="none" style:position="absolute" style:top="50%" style:left="50%" style:transform="translate(-50%, -50%)" style:width="16px" style:height="2px" style:backdrop-filter="invert()"></div>
        <div style:pointer-events="none" style:position="absolute" style:top="50%" style:left="50%" style:transform="translate(-50%, -50%)" style:width="2px" style:height="16px" style:backdrop-filter="invert()"></div>
    {/if}
</div>

<span>FPS: {displayedFPS}</span>
<p>
    Camera position: {camera.x}, {camera.y}, {camera.z}
</p>

<style>
</style>