<script lang="ts">
	import type { Snippet } from 'svelte';
	let {
		initialX = 0,
		initialY = 0,
		children
	} = $props<{
		initialX?: number;
		initialY?: number;
		children: Snippet;
	}>();
	let x: number = $state(initialX);
	let y: number = $state(initialY);

	let down: boolean = false;

	function onmousemove(e: MouseEvent) {
		e.preventDefault();
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

<div {onmousedown} class="absolute" style:top="{y}px" style:left="{x}px" role="group">
	{@render children()}
</div>

<svelte:window {onmousemove} {onmouseup} />
