<script lang="ts">
	import Draggable from '$lib/components/draggable.svelte';
	import Picture from '$lib/components/picture.svelte';

	let pictures: string[] = $state([]);

	function onchange(e: Event & { currentTarget: HTMLInputElement }) {
		const files = e.currentTarget.files;
		if (files != null) {
			[...files].map((file) => {
				if (file.type.includes('image')) {
					const src = URL.createObjectURL(file);
					pictures.push(src);
				}
			});
		}
	}
</script>

<head>
	<title>schizoboard</title>
</head>

<!-- TODO: drag and drop upload (handle file and picture) -->

<Draggable>
	<h1>Hello diddy</h1>
</Draggable>

<Picture></Picture>

{#if pictures.length != 0}
	{#each pictures as picture}
		<Picture src={picture}></Picture>
	{/each}
{/if}

<!-- TODO: add styling to upload button -->
<input type="file" id="upload" {onchange} hidden />
<label for="upload">Upload File</label>
