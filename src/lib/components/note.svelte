<script lang="ts">
	import Draggable from "./draggable.svelte";

	let { pos = $bindable({ x: 0, y: 0 }) } = $props<{
		pos: { x: number; y: number };
	}>();
	let text = $state("Test");

	let down = $state(false);
	let doubleclick = $state(false);

	function onblur() {
		doubleclick = false;
	}
</script>

<div class="relative bg-[#bada55]">
	<Draggable bind:pos bind:down bind:doubleclick>
		{#if !doubleclick}
			<p>{text}</p>
		{:else}
			<textarea
				bind:value={text}
				autofocus
				{onblur}
				onkeydown={(e) => {
					if (e.key === "Enter") {
						(e.target as HTMLInputElement).blur();
					}
				}}
			>
			</textarea>
			<p></p>
		{/if}
	</Draggable>
</div>
