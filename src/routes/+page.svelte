<script lang="ts">
	import Draggable from '$lib/components/draggable.svelte';
	import Picture from '$lib/components/picture.svelte';
	import Note from '$lib/components/note.svelte';
	import Document from '$lib/components/document.svelte';

	type entity = { type: 'note' | 'picture' | 'document'; src?: string };

	let entities: entity[] = $state([{ type: 'note' }]);
	let pos: { x: number; y: number } = $state({ x: 0, y: 0 });
	let focused: number | undefined = $state();

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
	}

	function onkeydown(e: KeyboardEvent) {
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

<div class="absolute h-full w-full" {ondragover} {ondrop} {onclick} role="main">
	{#each entities as entity, i}
		{#if entity.type == 'picture'}
			<Picture bind:focused bind:pos src={entity.src} dragged={true} index={i} />
		{:else if entity.type == 'note'}
			<Note bind:focused bind:pos index={i} />
		{:else if entity.type == 'document'}
			<Document bind:focused bind:pos type="pdf" src={entity.src!} index={i} />
		{/if}
	{/each}

	<!-- TODO: add styling to upload button -->
	<input type="file" id="upload" {onchange} hidden />
	<label for="upload">Upload File</label>
</div>

<svelte:window {onkeydown} {onmousemove} />
