<script lang="ts">
	import type { Snippet } from 'svelte';

	let {
		initialX = 0,
		initialY = 0,
		children,
		down = $bindable(false),
		doubleclick = $bindable(false),
		pos = $bindable({ x: 0, y: 0 }),
		index,
		focused = $bindable(),
		zoom = $bindable(1)
	} = $props<{
		initialX?: number;
		initialY?: number;
		children: Snippet;
		down?: boolean;
		doubleclick?: boolean;
		pos: { x: number; y: number };
		index: number;
		focused: number | undefined;
		zoom: number;
	}>();
	let x: number = $state(initialX);
	let y: number = $state(initialY);

	let offsetX = 0;
	let offsetY = 0;

	$effect(() => {
		if (down) {
			const mouseX = pos.x / zoom;
			const mouseY = pos.y / zoom;

			x = mouseX + offsetX;
			y = mouseY + offsetY;
		}
	});

	function onmousedown(e: MouseEvent) {
		offsetX = x - e.clientX / zoom;
		offsetY = y - e.clientY / zoom;
		down = true;
	}

	function onmouseup() {
		down = false;
	}

	function ondblclick() {
		doubleclick = true;
	}

	function onclick(e: MouseEvent) {
		e.stopPropagation();
		focused = index;
		//console.log(focused);
	}
</script>

<div
	{onclick}
	{onmousedown}
	{ondblclick}
	class="absolute select-none"
	style="transform: translate({x}px, {y}px);"
>
	{#if down}
		<div class="absolute inset-0 z-10 pointer-events-none" />
	{/if}

	<img src="/src/lib/assets/pushpin-down.png" class="relative w-10 h-auto mx-auto left-3 top-6 z-999">
	{@render children()}
</div>

<svelte:window {onmouseup} />
