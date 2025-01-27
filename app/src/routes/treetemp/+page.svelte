<script lang="ts">
	import Treetemp from './treetemp.svelte';
</script>

<div class="workbase">
	<div class="head"></div>
	<div class="main bg-orange-100">
		<Treetemp />
	</div>
	<div class="footer">footer</div>
</div>

<style>
	.workbase {
		@apply fixed inset-0 m-1 flex-col bg-green-300 p-0;
		min-height: 99vh;
	}

	.head,
	.footer {
		@apply h-[5%] bg-blue-100;
	}
	.main {
		@apply h-[89%];
	}
</style>
