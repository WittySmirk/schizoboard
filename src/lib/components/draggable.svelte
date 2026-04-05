<script lang="ts">
	import type { Snippet } from 'svelte';

	let {
		initialPos,
		children,
		down = $bindable(false),
		doubleclick = $bindable(false),
		pos = $bindable({ x: 0, y: 0 }),
		index,
		focused = $bindable(),
		zoom = $bindable(1),
		type,
		aspectRatio
	} = $props<{
		initialPos: { x: number; y: number; w: number; h: number };
		children: Snippet;
		down?: boolean;
		doubleclick?: boolean;
		pos: { x: number; y: number };
		index: number;
		focused: number | undefined;
		zoom: number;
		type: 'document' | 'note' | 'picture';
		aspectRatio?: number;
	}>();

	let object: { x: number; y: number; w: number; h: number } = $state(initialPos);

	let resizing = $state(false);
	let resizeStart: { x: number; y: number; w: number; h: number } | undefined = undefined;

	let offsetX = 0;
	let offsetY = 0;

	let zCounter = $state(1);

	$effect(() => {
		if (resizing && resizeStart != undefined) {
			object.w = resizeStart.w + (pos.x - resizeStart.x) / zoom;
			object.h = aspectRatio
				? object.w / aspectRatio
				: resizeStart.h + (pos.y - resizeStart.y) / zoom;
		}
		if (down && !resizing) {
			const mouseX = pos.x / zoom;
			const mouseY = pos.y / zoom;
			object.x = mouseX + offsetX;
			object.y = mouseY + offsetY;
		}
	});

	function onmousedown(e: MouseEvent) {
		offsetX = object.x - e.clientX / zoom;
		offsetY = object.y - e.clientY / zoom;
		down = true;
	}

	function onmouseup() {
		down = false;
		resizing = false;
	}

	function ondblclick() {
		doubleclick = true;
	}

	function onclick(e: MouseEvent) {
		e.stopPropagation();
		focused = index;
		zCounter++;
		console.log(zCounter);
	}

	function resizedown(e: MouseEvent) {
		e.stopPropagation();
		resizeStart = { x: e.clientX, y: e.clientY, w: object.w, h: object.h };
		resizing = true;
	}

	
</script>

<div
	{onclick}
	{onmousedown}
	{ondblclick}
	class="absolute"
	style="z-index:{zCounter}; transform: translate({object.x}px, {object.y}px); width: {object.w}px; height: {object.h}px;">

	<img src="/src/lib/assets/pushpin-down.png" class="fixed w-10 h-auto left-1/2 -translate-x-1/2 -top-5 z-[999] select-none ">
	{@render children()}

	{#if type != 'document'}
		<div
			onmousedown={resizedown}
			class="absolute right-0 bottom-0 h-1 h-4 w-1 w-4 cursor-se-resize bg-blue-500 opacity-0 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.2)]"
		>
		</div>
	{/if}
</div>

<svelte:window {onmouseup} />
