<script lang="ts">
	import type { Snippet } from 'svelte';
	let {
		initialX = 0,
		initialY = 0,
		children,
		down = $bindable(false),
		doubleclick = $bindable(false)
	} = $props<{
		initialX?: number;
		initialY?: number;
		children: Snippet;
		down?: boolean;
		doubleclick?: boolean;
	}>();
	let x: number = $state(initialX);
	let y: number = $state(initialY);

	let offsetX = 0;
	let offsetY = 0;

	function onmousemove(e: MouseEvent) {
		e.preventDefault();
		if (down) {
			x = e.clientX + offsetX;
			y = e.clientY + offsetY;
		}
	}
	function onmousedown(e: MouseEvent) {
		offsetX = x - e.clientX;
		offsetY = y - e.clientY;
		down = true;
	}

	function onmouseup() {
		down = false;
	}

	function ondblclick () {
		doubleclick = true;
		
	}
</script>

<div {onmousedown} {ondblclick} class="absolute" style:top="{y}px" style:left="{x}px">
	{@render children()}
</div>

<svelte:window {onmousemove} {onmouseup} />
