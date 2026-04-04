<script lang="ts">
	let { create = $bindable() } = $props<{
		create: 'note' | 'picture' | 'document' | undefined;
	}>();
</script>

<div
	class="fixed bottom-10 left-1/4 z-50 flex h-12 w-1/2 justify-around rounded-lg bg-[#646D74] text-white"
>
	<button type="button" onclick={() => (create = 'note')}>
		<img src="src/lib/assets/sticky_icon.svg" class="h-10 w-10" />
	</button>
	<button type="button" onclick={() => (create = 'picture')}>Picture</button>
	<button type="button" onclick={() => (create = 'document')}>Document</button>
</div>
