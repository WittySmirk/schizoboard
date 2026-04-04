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

	let object: { x: number; y: number; w: number; h: number } = $state({
		x: initialX,
		y: initialY,
		w: 200,
		h: 200
	});

	let resizing = $state(false);
	let resizeStart: { x: number; y: number; w: number; h: number } | undefined = undefined;

	let offsetX = 0;
	let offsetY = 0;

	$effect(() => {
		if (resizing && resizeStart != undefined) {
			object.w = resizeStart.w + (pos.x - resizeStart.x) / zoom;
			object.h = resizeStart.h + (pos.y - resizeStart.y) / zoom;
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
		//console.log(focused);
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
	class="absolute h-48 w-48"
	style="transform: translate({object.x}px, {object.y}px); width: {object.w}px; height: {object.h}px;"
>
	{@render children()}
	<div
		onmousedown={resizedown}
		class="absolute right-0 bottom-0 h-1 w-1 cursor-se-resize bg-blue-500"
	>
		a
	</div>
</div>

<svelte:window {onmouseup} />
