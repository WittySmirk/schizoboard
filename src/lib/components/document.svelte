<script lang="ts">
	import Draggable from './draggable.svelte';
	import { PdfViewer, PdfRenderer } from 'svelte-pdf-view';

	let {
		pos = $bindable({ x: 0, y: 0 }),
		type,
		src
	} = $props<{
		pos: { x: number; y: number };
		type: 'pdf' | 'md';
		src: string;
	}>();

	let down = $state(false);
</script>

<Draggable bind:pos bind:down>
	<div class="h-205">
		<PdfViewer {src}>
			<PdfRenderer backgroundColor="#FFFFFF" />
		</PdfViewer>
	</div>
</Draggable>
