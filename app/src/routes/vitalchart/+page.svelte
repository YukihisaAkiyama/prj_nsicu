<script lang="ts">
	import ChartVital from './chart_vital.svelte';
	import EventList from './event_list.svelte';
</script>

<div class="workbase">
	<div class="head"></div>
	<div class="main bg-orange-100">
		<div class="chart h-[30%] overflow-auto">
			<ChartVital />
		</div>
		<div class="event h-[70%] overflow-x-auto overflow-y-auto">
			<EventList />
		</div>
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
