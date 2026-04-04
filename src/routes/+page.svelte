<script lang="ts" module>
	export type entity = {
		type: 'note' | 'picture' | 'document';
		src?: string;
		initial?: { x: number; y: number };
	};
</script>

<script lang="ts">
	import Picture from '$lib/components/picture.svelte';
	import Note from '$lib/components/note.svelte';
	import Document from '$lib/components/document.svelte';
	import Toolbar from '$lib/components/toolbar.svelte';

	// TODO: add actual drag checking
	let entities: entity[] = $state([{ type: 'note', initial: { x: 0, y: 0 } }]);
	let pos: { x: number; y: number } = $state({ x: 0, y: 0 });
	let focused: number | undefined = $state();

	let zoom = $state(1);
	const ZOOM_FACTOR = 0.05;
	let scrollVal = $state(0);
	let isPanning = false;
	let lastPan = $state({ x: 0, y: 0 });
	let offset = $state({ x: 0, y: 0 });
	let drop: { x: number; y: number } | undefined = $state();

	let create: 'note' | 'picture' | 'document' | undefined;

	function parseFiles(files: FileList, initial?: { x: number; y: number }) {
		[...files].map((file) => {
			console.log(file.type);
			if (file.type.includes('image')) {
				const src = URL.createObjectURL(file);
				entities.push({ type: 'picture', src, initial });
			}
			if (file.type.includes('pdf')) {
				const src = URL.createObjectURL(file);
				entities.push({ type: 'document', src, initial });
			}
		});
	}

	function onchange() {}
	function ondragover(e: Event) {
		e.preventDefault();
	}

	function ondrop(e: DragEvent) {
		e.preventDefault();
		if (e.dataTransfer != null && e.dataTransfer.files) {
			//pos = { x: (e.clientX - offset.x) / zoom, y: (e.clientY - offset.y) / zoom };
			drop = { x: (e.clientX - offset.x) / zoom, y: (e.clientY - offset.y) / zoom };
			const files = e.dataTransfer.files;
			parseFiles(files);
		}
	}

	function onclick(e: MouseEvent) {
		if (create != undefined) {
			switch (create) {
				case 'note':
					entities.push({
						type: 'note',
						initial: { x: (e.clientX - offset.x) / zoom, y: (e.clientY - offset.y) / zoom }
					});
					break;
				case 'picture':
					{
						const input: HTMLInputElement = document.createElement('input');
						input.type = 'file';
						input.accept = 'image/*';
						input.onchange = (ev: Event) => {
							const files = (ev.currentTarget as HTMLInputElement).files;
							if (files != null) {
								parseFiles(files, {
									x: (e.clientX - offset.x) / zoom,
									y: (e.clientY - offset.y) / zoom
								});
							}
						};
						input.click();
					}
					break;
				case 'document':
					{
						const input: HTMLInputElement = document.createElement('input');
						input.type = 'file';
						input.accept = 'application/pdf';
						input.onchange = (ev: Event) => {
							const files = (ev.currentTarget as HTMLInputElement).files;
							if (files != null) {
								parseFiles(files, {
									x: (e.clientX - offset.x) / zoom,
									y: (e.clientY - offset.y) / zoom
								});
							}
						};
						input.click();
					}
					break;
			}
			create = undefined;
		}
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

<div
	class="fixed inset-0"
	style="transform-origin: 0 0; transform: translate({offset.x}px, {offset.y}px) scale({zoom})"
>
	<canvas
		class="fixed -z-9999 h-full w-full bg-[url(/src/lib/assets/corkboard.jpg)] bg-size-[200px] bg-repeat inset-shadow-[0_0_200px_rgba(0,0,0,0.9)] brightness-95"
		style="transform-origin: 0 0; transform: scale({1 /
			zoom}) translate({-offset.x}px, {-offset.y}px);
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
			<Picture
				bind:focused
				bind:pos
				bind:zoom
				src={entity.src}
				index={i}
				{drop}
				initialPos={entity.initial}
			/>
		{:else if entity.type == 'note'}
			<Note bind:focused bind:pos bind:zoom index={i} initialPos={entity.initial!} />
		{:else if entity.type == 'document'}
			<Document
				bind:focused
				bind:pos
				bind:zoom
				type="pdf"
				src={entity.src!}
				index={i}
				{drop}
				initialPos={entity.initial}
			/>
		{/if}
	{/each}

	<!-- TODO: add styling to upload button -->
	<!--
	<input type="file" id="upload" {onchange} hidden />
	<label for="upload">Upload File</label>
	-->
</div>

<Toolbar bind:create />
