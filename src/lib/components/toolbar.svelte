<script lang="ts">
	let { create = $bindable(), zoom = $bindable(1)} = $props<{
		create: 'note' | 'picture' | 'document' | undefined;
		zoom: number;
	}>();

	let dragging =  $state<'note' | 'picture' | 'document' | null>(null);
	let mouse = $state({ x: 0, y: 0 });


	function onmousemove(e: MouseEvent) {
		mouse.x = e.clientX;
		mouse.y = e.clientY;
	}

	function startDrag(type: 'note' | 'picture' | 'document') {
		dragging = type;
		console.log(dragging);
	}

	function onclick(e: MouseEvent) {
		if ((e.target as HTMLElement).closest('.sidebar-menu')) {
			console.log("ignored");
			return;
		}

		if (dragging) {
			create = dragging;
			console.log(dragging + " created");
			dragging = null;
		}
	}
</script>

<svelte:window {onmousemove} {onclick} />

<div
	class="sidebar-menu fixed z-9999 flex h-full w-60 bg-[url(/src/lib/assets/woodtexture2.png)] flex-col pl-6 justify-center gap-15 bg-cover drop-shadow-[19px_10px_21px_rgba(0,0,0,0.5)] brightness-95" style="filter: drop-shadow(19px 10px 21px rgba(0,0,0,{0.5*zoom}))"
>
	<button type="button" onclick={(e) => (startDrag('note'))}>
		<img src="src/lib/assets/stickynote-pile.png" class="relative h-35 w-35 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)] saturate-40 brightness-95" />
	</button>
	<button type="button" onclick={(e) => (startDrag('picture'))}>
		<div class="relative h-45 w-35 bg-[#ffffed] brightness-95 p-2 overflow-hidden rotate-3 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)]" >
			<img src="src/lib/assets/placeholder.png" class="w-full h-7/8 object-cover brightness-50 saturate-0 contrast-200"/> 
		</div>
	</button>
	<button type="button" onclick={(e) => (startDrag('document'))}>
		<div class="relative h-45 w-35 bg-[url(/images/stack-papers.png)] bg-cover p-6 brightness-95 overflow-hidden -rotate-2 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)]" >
			<img src="/images/document-placeholder.png" class="w-7/8 h-7/8 mt-1 ml-4 mx-auto object-cover -rotate-1 brightness-50 saturate-0 contrast-75"/> 
		</div>
	</button>
</div>

{#if dragging}
	<div
		class="fixed pointer-events-none z-10000 opacity-50 scale-75"
		style="left: {mouse.x}px; top: {mouse.y}px; transform: translate(-10%, -10%);"
	>
		{#if dragging == 'note'}
			<img src="src/lib/assets/stickynote.png" class="w-24" />
		{:else if dragging == 'picture'}
			<img src="src/lib/assets/placeholder.png" class="w-24" />
		{:else if dragging == 'document'}
			<img src="/images/document-placeholder.png" class="w-24" />
		{/if}
	</div>
{/if}