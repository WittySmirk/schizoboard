<script lang="ts">
	import Draggable from '$lib/components/draggable.svelte';
	import Picture from '$lib/components/picture.svelte';
	import Note from '$lib/components/note.svelte';

	let pictures: string[] = $state([]);

	function parseFiles(files: FileList) {
		[...files].map((file) => {
			if (file.type.includes('image')) {
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
</script>

<head>
	<title>schizoboard</title>
</head>

<div class="absolute h-full w-full" {ondragover} {ondrop} role="main">
	<Draggable>
		<h1>Hello diddy</h1>
	</Draggable>

	<Picture></Picture>

	{#if pictures.length != 0}
		{#each pictures as picture}
			<Picture src={picture}></Picture>
		{/each}
	{/if}

	<Note></Note>

	<!-- TODO: add styling to upload button -->
	<input type="file" id="upload" {onchange} hidden />
	<label for="upload">Upload File</label>
</div>
