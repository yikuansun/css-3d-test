<script lang="ts">
	let { children, perspective, camera } = $props();
</script>

<div style:width="100%" style:height="100%" style:position="absolute" style:transform-style="preserve-3d"
    style:transform="translateZ({perspective}px)">
    <div style:width="100%" style:height="100%" style:position="absolute" style:transform-style="preserve-3d"
        style:transform="rotateX({camera.angleX}deg) rotateY({camera.angleY}deg) rotateZ({camera.angleZ}deg)">
        <div style:width="100%" style:height="100%" style:position="absolute" style:transform-style="preserve-3d"
            style:transform="translateX({-camera.x}px) translateY({-camera.y}px) translateZ({-camera.z}px) ">
            {@render children()}
        </div>
    </div>
</div>