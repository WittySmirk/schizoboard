<script lang="ts">
	import Draggable from "$lib/components/draggable.svelte";
	import Picture from "$lib/components/picture.svelte";
	import Note from "$lib/components/note.svelte";

	let pictures: string[] = $state([]);
	let pos: { x: number; y: number } = $state({ x: 0, y: 0 });

	function parseFiles(files: FileList) {
		[...files].map((file) => {
			if (file.type.includes("image")) {
				const src = URL.createObjectURL(file);
				pictures.push(src);
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
			const files = e.dataTransfer.files;
			parseFiles(files);
		}
	}

	function onmousemove(e: MouseEvent) {
		e.preventDefault();
		pos = { x: e.clientX, y: e.clientY };
	}
</script>

<head>
	<title>schizoboard</title>
</head>

<div class="absolute h-full w-full" {ondragover} {ondrop} role="main">
	<Draggable bind:pos>
		<h1>Hello diddy</h1>
	</Draggable>

	<Picture bind:pos></Picture>

	{#if pictures.length != 0}
		{#each pictures as picture}
			<Picture bind:pos src={picture}></Picture>
		{/each}
	{/if}

	<Note bind:pos></Note>

	<!-- TODO: add styling to upload button -->
	<input type="file" id="upload" {onchange} hidden />
	<label for="upload">Upload File</label>
</div>

<svelte:window {onmousemove} />
