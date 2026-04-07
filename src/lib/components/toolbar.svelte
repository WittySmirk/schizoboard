<script lang="ts">
	let { create = $bindable(), zoom = $bindable(1) } = $props<{
		create: 'note' | 'picture' | 'document' | undefined;
		zoom: number;
	}>();

	let dragging = $state<'note' | 'picture' | 'document' | null>(null);
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
			console.log('ignored');
			return;
		}

		if (dragging) {
			create = dragging;
			console.log(dragging + ' created');
			dragging = null;
		}
	}
</script>

<svelte:window {onmousemove} {onclick} />

<div
	class="sidebar-menu fixed z-9999 flex h-full w-60 flex-col justify-center gap-15 bg-[url(/images/woodtexture2.png)] bg-cover pl-6 brightness-95 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.5)]"
	style="filter: drop-shadow(19px 10px 21px rgba(0,0,0,{0.5 * zoom}))"
>
	<button type="button" class="hover:cursor-pointer" onclick={(e) => startDrag('note')}>
		<img
			src="/images/stickynote-pile.png"
			class="relative h-35 w-35 brightness-95 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)] saturate-40"
		/>
	</button>
	<button type="button" class="hover:cursor-pointer" onclick={(e) => startDrag('picture')}>
		<div
			class="relative h-45 w-35 rotate-3 overflow-hidden bg-[#ffffed] p-2 brightness-95 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)] hover:cursor-pointer"
		>
			<img
				src="/images/placeholder.png"
				class="h-7/8 w-full object-cover brightness-50 contrast-200 saturate-0"
			/>
		</div>
	</button>
	<button type="button" class="hover:cursor-pointer" onclick={(e) => startDrag('document')}>
		<div
			class="relative h-45 w-35 -rotate-2 overflow-hidden bg-[url(/images/stack-papers.png)] bg-cover p-6 brightness-95 drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)] hover:cursor-pointer"
		>
			<img
				src="/images/document-placeholder.png"
				class="mx-auto mt-1 ml-4 h-7/8 w-7/8 -rotate-1 object-cover brightness-50 contrast-75 saturate-0"
			/>
		</div>
	</button>
</div>

{#if dragging}
	<div
		class="pointer-events-none fixed z-10000 scale-75 opacity-50"
		style="left: {mouse.x}px; top: {mouse.y}px; transform: translate(-10%, -10%);"
	>
		{#if dragging == 'note'}
			<img src="/images/stickynote.png" class="w-24" />
		{:else if dragging == 'picture'}
			<img src="/images/placeholder.png" class="w-24" />
		{:else if dragging == 'document'}
			<img src="/images/document-placeholder.png" class="w-24" />
		{/if}
	</div>
{/if}

