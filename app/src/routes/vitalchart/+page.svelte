<script lang="ts">
	import ChartVital from './chart_vital.svelte';
	import EventList from './event_list.svelte';
	import Test1 from './test1.svelte';
	import Test2 from './test2.svelte';
</script>

<div class="workbase">
	<div class="head"></div>
	<div class="main bg-orange-100">
		<div class="chart h-[300px] overflow-auto">
			<ChartVital />
		</div>
		<div class="event h-[300px] overflow-auto">
			<EventList />
		</div>
		<div class="test1 h-[300px] overflow-auto">
			<Test2 />
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
