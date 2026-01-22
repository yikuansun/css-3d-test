<script lang="ts">
    import Plane from "$lib/components/Plane.svelte";
    import World from "$lib/components/World.svelte";
	import { onDestroy, onMount } from "svelte";

    let camera = $state({
        x: 0,
        y: -50,
        z: 0,
        angleX: 0,
        angleY: 0,
        angleZ: 0,
        xOffset: 480,
        yOffset: 270,
    });

    let playerVelocity = $state([0, 0, 0]); // x, y, z

    const TEXTURE_NAMES: { [key: number]: string } = {
        1: "grass",
        2: "dirt",
    };

    const CUBE_SIZE = 48;

    const perspective = 1024;

    let keysPressed: { [key: string]: boolean } = {};

    const TRANSLATION_SPEED = 200; // px / sec
    const ROTATION_SPEED = 150; // deg / sec

    const GRAVITY = 800;
    const JUMP_SPEED = 500;

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

        playerVelocity[0] = 0;
        playerVelocity[2] = 0;
        if (keysPressed["w"]) {
            playerVelocity[2] -= TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180);
            playerVelocity[0] += TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180);
        }
        if (keysPressed["s"]) {
            playerVelocity[2] += TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180);
            playerVelocity[0] -= TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180);
        }
        if (keysPressed["a"]) {
            playerVelocity[0] -= TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180);
            playerVelocity[2] -= TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180);
        }
        if (keysPressed["d"]) {
            playerVelocity[0] += TRANSLATION_SPEED * Math.cos(camera.angleY * Math.PI / 180);
            playerVelocity[2] += TRANSLATION_SPEED * Math.sin(camera.angleY * Math.PI / 180);
        }

        let blockBeneathPlayer = worldPositionToGridCoord(camera.x, camera.y + 3 * CUBE_SIZE / 2 + 5, camera.z);
        if (worldMatrix?.[blockBeneathPlayer[0]]?.[blockBeneathPlayer[1]]?.[blockBeneathPlayer[2]] !== 0) {
            //playerVelocity[1] = 0;
            if (keysPressed[" "]) {
                playerVelocity[1] = -JUMP_SPEED;
            }
        }

        playerVelocity[1] += GRAVITY * deltaTime;

        camera.x += playerVelocity[0] * deltaTime;
        camera.y += playerVelocity[1] * deltaTime;
        camera.z += playerVelocity[2] * deltaTime;
        let playerXEdges = [
            worldPositionToGridCoord(camera.x - CUBE_SIZE / 2, camera.y, camera.z),
            worldPositionToGridCoord(camera.x + CUBE_SIZE / 2, camera.y, camera.z),
            worldPositionToGridCoord(camera.x - CUBE_SIZE / 2, camera.y + CUBE_SIZE, camera.z),
            worldPositionToGridCoord(camera.x + CUBE_SIZE / 2, camera.y + CUBE_SIZE, camera.z),
        ];
        let playerYEdges = [
            worldPositionToGridCoord(camera.x, camera.y - CUBE_SIZE / 2, camera.z),
            worldPositionToGridCoord(camera.x, camera.y + 3 * CUBE_SIZE / 2, camera.z),
        ];
        let playerZEdges = [
            worldPositionToGridCoord(camera.x, camera.y, camera.z - CUBE_SIZE / 2),
            worldPositionToGridCoord(camera.x, camera.y, camera.z + CUBE_SIZE / 2),
            worldPositionToGridCoord(camera.x, camera.y + CUBE_SIZE, camera.z - CUBE_SIZE / 2),
            worldPositionToGridCoord(camera.x, camera.y + CUBE_SIZE, camera.z + CUBE_SIZE / 2),
        ];
        if (playerXEdges[0][0] < 0 || playerXEdges[1][0] >= worldMatrix.length || worldMatrix?.[playerXEdges[0][0]]?.[playerXEdges[0][1]]?.[playerXEdges[0][2]] !== 0 || worldMatrix?.[playerXEdges[1][0]]?.[playerXEdges[1][1]]?.[playerXEdges[1][2]] !== 0 || worldMatrix?.[playerXEdges[2][0]]?.[playerXEdges[2][1]]?.[playerXEdges[2][2]] !== 0 || worldMatrix?.[playerXEdges[3][0]]?.[playerXEdges[3][1]]?.[playerXEdges[3][2]] !== 0) {
            camera.x -= playerVelocity[0] * deltaTime;
        }
        if (playerYEdges[0][1] < 0 || playerYEdges[1][1] >= worldMatrix[0].length || worldMatrix?.[playerYEdges[0][0]]?.[playerYEdges[0][1]]?.[playerYEdges[0][2]] !== 0 || worldMatrix?.[playerYEdges[1][0]]?.[playerYEdges[1][1]]?.[playerYEdges[1][2]] !== 0) {
            camera.y -= playerVelocity[1] * deltaTime;
            playerVelocity[1] = 0;
        }
        if (playerZEdges[0][2] < 0 || playerZEdges[1][2] >= worldMatrix[0][0].length || worldMatrix?.[playerZEdges[0][0]]?.[playerZEdges[0][1]]?.[playerZEdges[0][2]] !== 0 || worldMatrix?.[playerZEdges[1][0]]?.[playerZEdges[1][1]]?.[playerZEdges[1][2]] !== 0 || worldMatrix?.[playerZEdges[2][0]]?.[playerZEdges[2][1]]?.[playerZEdges[2][2]] !== 0 || worldMatrix?.[playerZEdges[3][0]]?.[playerZEdges[3][1]]?.[playerZEdges[3][2]] !== 0) {
            camera.z -= playerVelocity[2] * deltaTime;
        }

        /*// camera rotation
        if (keysPressed["ArrowLeft"]) camera.angleY -= ROTATION_SPEED * deltaTime;
        if (keysPressed["ArrowRight"]) camera.angleY += ROTATION_SPEED * deltaTime;
        if (keysPressed["ArrowUp"]) camera.angleX += ROTATION_SPEED * deltaTime;
        if (keysPressed["ArrowDown"]) camera.angleX -= ROTATION_SPEED * deltaTime;*/
        
        scanBlockFocus();

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

    function gridCoordToBlockPosition(layerX: number, layerY: number, layerZ: number) {
        return {
            x: (layerX - worldMatrix.length / 2) * CUBE_SIZE,
            y: (layerY - worldMatrix.length / 2) * CUBE_SIZE,
            z: (layerZ - worldMatrix.length / 2) * CUBE_SIZE,
        };
    }

    function worldPositionToGridCoord(x: number, y: number, z: number) {
        return [
            Math.round(x / CUBE_SIZE + worldMatrix.length / 2),
            Math.round(y / CUBE_SIZE + worldMatrix.length / 2),
            Math.round(z / CUBE_SIZE + worldMatrix.length / 2),
        ];
    }

    let focusedBlockPosition = $state([0, 0, 0]);
    const focusedBlockStyle = "outline: 1px solid white; outline-offset: -1px;";
    let placingBlockPosition = $state([0, 0, 0]);

    function scanBlockFocus() {
        const radX = camera.angleX * Math.PI / 180;
        const radY = camera.angleY * Math.PI / 180;
        const dirX = Math.cos(radX) * Math.sin(radY);
        const dirY = - Math.sin(radX);
        const dirZ = - Math.cos(radX) * Math.cos(radY);

        let focusedBlockWorldPosition = [camera.x, camera.y, camera.z];
        focusedBlockPosition = worldPositionToGridCoord(focusedBlockWorldPosition[0], focusedBlockWorldPosition[1], focusedBlockWorldPosition[2]);
        const STEP_SIZE = CUBE_SIZE / 4;
        while (worldMatrix?.[focusedBlockPosition[0]]?.[focusedBlockPosition[1]]?.[focusedBlockPosition[2]] === 0) {
            focusedBlockWorldPosition[0] += STEP_SIZE * dirX;
            focusedBlockWorldPosition[1] += STEP_SIZE * dirY;
            focusedBlockWorldPosition[2] += STEP_SIZE * dirZ;
            focusedBlockPosition = worldPositionToGridCoord(focusedBlockWorldPosition[0], focusedBlockWorldPosition[1], focusedBlockWorldPosition[2]);
        }
        placingBlockPosition = worldPositionToGridCoord(
            focusedBlockWorldPosition[0] - STEP_SIZE * dirX,
            focusedBlockWorldPosition[1] - STEP_SIZE * dirY,
            focusedBlockWorldPosition[2] - STEP_SIZE * dirZ,
        );
    }

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
    onclick={(e) => {
        if (!document.pointerLockElement) {
            document.body.requestPointerLock();
        }
    }}
    onmousedown={(e) => {
        if (document.pointerLockElement) {
            if (e.button === 0) {
                // mine
                if (worldMatrix?.[focusedBlockPosition[0]]?.[focusedBlockPosition[1]]?.[focusedBlockPosition[2]] !== 0) {
                    worldMatrix[focusedBlockPosition[0]][focusedBlockPosition[1]][focusedBlockPosition[2]] = 0;
                }
            }
            else if (e.button === 2) {
                console.log("right clik", placingBlockPosition)
                if (worldMatrix?.[placingBlockPosition[0]]?.[placingBlockPosition[1]]?.[placingBlockPosition[2]] === 0)
                    worldMatrix[placingBlockPosition[0]][placingBlockPosition[1]][placingBlockPosition[2]] = 1;
            }
        }
    }}
    onmousemove={(e) => {
        if (document.pointerLockElement) {
            camera.angleY += (e.movementX / 1000) * ROTATION_SPEED;
            camera.angleX -= (e.movementY / 1000) * ROTATION_SPEED;

            if (camera.angleX > 90) camera.angleX = 90;
            if (camera.angleX < -90) camera.angleX = -90;
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
                    {@const { x: cubeX, y: cubeY, z: cubeZ } = gridCoordToBlockPosition(layerX, layerY, layerZ)}
                    {#if cell > 0}
                        {#if worldMatrix?.[layerX]?.[layerY - 1]?.[layerZ] === 0}
                            <!-- Top -->
                            <Plane x={cubeX} y={cubeY - CUBE_SIZE / 2} z={cubeZ} angleX={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_top.png" brightness={120}
                                style={focusedBlockPosition[0] === layerX && focusedBlockPosition[1] === layerY && focusedBlockPosition[2] === layerZ ? focusedBlockStyle : ""} />
                        {/if}
                        {#if worldMatrix?.[layerX]?.[layerY + 1]?.[layerZ] === 0}
                            <!-- Bottom -->
                            <Plane x={cubeX} y={cubeY + CUBE_SIZE / 2} z={cubeZ} angleX={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_bottom.png" brightness={40}
                                style={focusedBlockPosition[0] === layerX && focusedBlockPosition[1] === layerY && focusedBlockPosition[2] === layerZ ? focusedBlockStyle : ""} />
                        {/if}
                        {#if worldMatrix?.[layerX - 1]?.[layerY]?.[layerZ] === 0}
                            <!-- Left -->
                            <Plane x={cubeX - CUBE_SIZE / 2} y={cubeY} z={cubeZ} angleY={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={60}
                                style={focusedBlockPosition[0] === layerX && focusedBlockPosition[1] === layerY && focusedBlockPosition[2] === layerZ ? focusedBlockStyle : ""} />
                        {/if}
                        {#if worldMatrix?.[layerX + 1]?.[layerY]?.[layerZ] === 0}
                            <!-- Right -->
                            <Plane x={cubeX + CUBE_SIZE / 2} y={cubeY} z={cubeZ} angleY={90} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={100}
                                style={focusedBlockPosition[0] === layerX && focusedBlockPosition[1] === layerY && focusedBlockPosition[2] === layerZ ? focusedBlockStyle : ""} />
                        {/if}
                        {#if worldMatrix?.[layerX]?.[layerY]?.[layerZ - 1] === 0}
                            <!-- Front -->
                            <Plane x={cubeX} y={cubeY} z={cubeZ - CUBE_SIZE / 2} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={80}
                                style={focusedBlockPosition[0] === layerX && focusedBlockPosition[1] === layerY && focusedBlockPosition[2] === layerZ ? focusedBlockStyle : ""} />
                        {/if}
                        {#if worldMatrix?.[layerX]?.[layerY]?.[layerZ + 1] === 0}
                            <!-- Back -->
                            <Plane x={cubeX} y={cubeY} z={cubeZ + CUBE_SIZE / 2} width={CUBE_SIZE} height={CUBE_SIZE} image="/block_textures/{TEXTURE_NAMES[cell]}_side.png" brightness={80}
                                style={focusedBlockPosition[0] === layerX && focusedBlockPosition[1] === layerY && focusedBlockPosition[2] === layerZ ? focusedBlockStyle : ""} />
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
    Camera position (world): {camera.x}, {camera.y}, {camera.z} <br />
    Camera position (block): {worldPositionToGridCoord(camera.x, camera.y, camera.z)} <br />
    Focused block position: {focusedBlockPosition}
</p>

<style>
</style>