<script lang="ts">
	import Draggable from '$lib/components/draggable.svelte';
	import Picture from '$lib/components/picture.svelte';
	import Note from '$lib/components/note.svelte';
	import Document from '$lib/components/document.svelte';

	type entity = { type: 'note' | 'picture' | 'document'; src?: string };

	// TODO: add actual drag checking
	let entities: entity[] = $state([{ type: 'note' }]);
	let pos: { x: number; y: number } = $state({ x: 0, y: 0 });
	let focused: number | undefined = $state();

	let zoom = $state(1);
	const ZOOM_FACTOR = 0.1;
	let scrollVal = $state(0);
	let isPanning = false;
	let lastPan = $state({ x: 0, y: 0 });
	let offset = $state({ x: 0, y: 0 });

	function parseFiles(files: FileList) {
		[...files].map((file) => {
			console.log(file.type);
			if (file.type.includes('image')) {
				const src = URL.createObjectURL(file);
				entities.push({ type: 'picture', src });
			}
			if (file.type.includes('pdf')) {
				const src = URL.createObjectURL(file);
				entities.push({ type: 'document', src });
			}
		});
	}

	function onchange(e: Event & { currentTarget: HTMLInputElement }) {
		const files = e.currentTarget.files;
		if (files != null) {
			parseFiles(files);
		}
	}
	function ondragover(e: Event) {
		e.preventDefault();
	}

	function ondrop(e: DragEvent) {
		e.preventDefault();
		if (e.dataTransfer != null && e.dataTransfer.files) {
			pos = { x: e.x, y: e.y };
			const files = e.dataTransfer.files;
			parseFiles(files);
		}
	}

	function onclick(e: MouseEvent) {
		focused = undefined;
	}

	function onmousemove(e: MouseEvent) {
		e.preventDefault();
		pos = { x: e.clientX, y: e.clientY };

		if (!isPanning) return;

		const dx = pos.x - lastPan.x;
		const dy = pos.y - lastPan.y;

		offset.x += dx;
		offset.y += dy;

		lastPan = { x: e.clientX, y: e.clientY };
		/*
		console.log(offset.x);
		console.log(offset.y);
		*/
	}

	function onwheel(e: WheelEvent) {
		zoom += e.deltaY > 0 ? -ZOOM_FACTOR : ZOOM_FACTOR;
	}

	function onmousedown(e: MouseEvent) {
		isPanning = true;
		lastPan = { x: e.clientX, y: e.clientY };
	}

	function onmouseup() {
		isPanning = false;
	}

	function onkeydown(e: KeyboardEvent) {
		console.log('here', focused);
		if (focused != undefined && e.key == 'Backspace') {
			console.log('removing at ', focused);
			entities.splice(focused, 1);
			entities = entities;
		}
	}
</script>

<head>
	<title>schizoboard</title>
</head>

<svelte:window {onwheel} bind:scrollY={scrollVal} {onmousemove} {onkeydown} />

<div class="fixed inset-0" style="transform: translate({offset.x}px, {offset.y}px) scale({zoom})">
	<canvas
		class="fixed -z-9999 h-full w-full bg-[url(/src/lib/assets/corkboard.jpg)] bg-size-[200px] bg-repeat inset-shadow-[0_0_200px_rgba(0,0,0,0.9)] brightness-95"
		style="transform: scale({1 / zoom}) translate({-offset.x}px, {-offset.y}px);
					background-size: {50 * zoom}%;
					background-position: {offset.x}px {offset.y}px;"
		{ondragover}
		{ondrop}
		{onmousedown}
		{onmouseup}
		{onclick}
	>
	</canvas>

	{#each entities as entity, i}
		{#if entity.type == 'picture'}
			<Picture bind:focused bind:pos bind:zoom src={entity.src} dragged={true} index={i} />
		{:else if entity.type == 'note'}
			<Note bind:focused bind:pos bind:zoom index={i} />
		{:else if entity.type == 'document'}
			<Document
				bind:focused
				bind:pos
				bind:zoom
				type="pdf"
				src={entity.src!}
				index={i}
				dragged={true}
			/>
		{/if}
	{/each}

	<!-- TODO: add styling to upload button -->
	<input type="file" id="upload" {onchange} hidden />
	<label for="upload">Upload File</label>
</div>

