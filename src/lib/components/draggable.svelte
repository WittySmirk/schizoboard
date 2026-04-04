<script lang="ts">
	let { initialX = 100, initialY = 100 } = $props();
	let x: number = $state(initialX);
	let y: number = $state(initialY);

	let down: boolean = false;

	function onmousemove(e) {
		if (down) {
			x = e.clientX;
			y = e.clientY;
		}
	}
	function onmousedown() {
		down = true;
	}
	function onmouseup() {
		down = false;
	}
</script>

<!-- TODO: props -->
<div {onmousedown} class="absolute bg-blue-500" style:top="{y}px" style:left="{x}px">
	<h1>hello diddy</h1>
</div>

<svelte:window {onmousemove} {onmouseup} />
