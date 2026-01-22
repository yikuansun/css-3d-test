<script>
    let { x, y, z = 0, angleX = 0, angleY = 0, angleZ = 0, width, height, color = "red", image = "", brightness = 100, text="", style="" } = $props();
</script>

<div
    style:position="absolute"
    style:top="{y - height / 2}px"
    style:left="{x - width / 2}px"
    style:transform="translateZ({z}px) rotateX({angleX}deg) rotateY({angleY}deg) rotateZ({angleZ}deg)"
    style:width="{width}px"
    style:height="{height}px"
    style:background-color="{color}"
    style:background-image="url('{image}')"
    style:background-size="cover"
    style:image-rendering="pixelated"
    style:filter="brightness({brightness}%)"
    style={style}
>{text}</div>