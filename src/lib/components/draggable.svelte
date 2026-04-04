<script lang="ts">
	import type { Snippet } from "svelte";
	let {
		initialX = 0,
		initialY = 0,
		children,
		down = $bindable(false),
		doubleclick = $bindable(false),
		pos = $bindable({ x: 0, y: 0 }),
	} = $props<{
		initialX?: number;
		initialY?: number;
		children: Snippet;
		down?: boolean;
		doubleclick?: boolean;
		pos: { x: number; y: number };
	}>();
	let x: number = $state(initialX);
	let y: number = $state(initialY);

	$effect(() => {
		if (down) {
			x = pos.x;
			y = pos.y;
		}
	});

	function onmousedown() {
		down = true;
	}

	function onmouseup() {
		down = false;
	}

	function ondblclick() {
		doubleclick = true;
	}
</script>

<div
	{onmousedown}
	{ondblclick}
	class="absolute"
	style="transform: translate({x}px, {y}px); will-change: transform; isolation: isolate;"
>
	{#if down}
		<div class="absolute inset-0 z-10" />
	{/if}
	{@render children()}
</div>

<svelte:window {onmousemove} {onmouseup} />
