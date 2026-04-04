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
		focused = $bindable()
	} = $props<{
		initialX?: number;
		initialY?: number;
		children: Snippet;
		down?: boolean;
		doubleclick?: boolean;
		pos: { x: number; y: number };
		index: number;
		focused: number | undefined;
	}>();
	let x: number = $state(initialX);
	let y: number = $state(initialY);

	let offsetX = 0;
	let offsetY = 0;

	$effect(() => {
		if (down) {
			x = pos.x + offsetX;
			y = pos.y + offsetY;
		}
	});

	function onmousedown(e: MouseEvent) {
		offsetX = x - e.clientX;
		offsetY = y - e.clientY;
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
	}
</script>

<div
	{onclick}
	{onmousedown}
	{ondblclick}
	class="absolute"
	style="transform: translate({x}px, {y}px);"
>
	{#if down}
		<div class="absolute inset-0 z-10" />
	{/if}
	{@render children()}
</div>

<svelte:window {onmouseup} />
