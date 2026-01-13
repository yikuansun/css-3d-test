<script lang="ts">
    import Plane from "$lib/components/Plane.svelte";
    import World from "$lib/components/World.svelte";

    const perspective = 300;
</script>

<div style:width="960px" style:height="540px" style:background-color="black"
    style:position="fixed" style:left="50vw" style:top="50vh"
    style:transform="translate(-50%, -50%) scale(calc(min(100vw / 960px, 100vh / 540px)))"
    style:perspective="{perspective}px">
    <World perspective={perspective}>
        <Plane x={480} y={270} z={-100} width={100} height={100} color="red" />
        <Plane x={480} y={270} z={-200} width={100} height={100} color="blue" />
        <Plane x={430} y={270} z={-150} angleY={90} width={100} height={100} color="green" />
        <Plane x={530} y={270} z={-150} angleY={90} width={100} height={100} color="yellow" />
    </World>
</div>

<style>
</style>