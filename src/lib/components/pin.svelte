<script lang="ts">
	let {
		onclick,
		pos = $bindable(),
		pinPos = $bindable(),
		zoom = $bindable()
	} = $props<{
		onclick: (e: MouseEvent) => void;
		pos: { x: number; y: number };
		zoom: number;
		pinPos: { x: number; y: number };
	}>();

	let pin: HTMLElement;
	$effect(() => {
		const _ = { pos, zoom };
		const rect = pin.getBoundingClientRect();
		pinPos = { x: rect.left + rect.width / 2, y: rect.top + rect.height / 2 };
	});
</script>

<div bind:this={pin} {onclick} class="absolute -top-5 left-1/2 h-10 w-10">a</div>
