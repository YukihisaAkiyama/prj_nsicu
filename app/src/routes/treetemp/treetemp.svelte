<script lang="ts">
	// カラーパレットの定義
	const colors = {
		normal: '#4CAF50', // 緑
		warning: '#FF5722', // オレンジ赤
		default: '#2196F3', // 青
		parent: '#9C27B0' // 紫
	};

	// フォントサイズの計算関数
	function calculateFontSize(d: any): string {
		const area = d.x1 - d.x0 * (d.y1 - d.y0);
		// 面積に応じて動的にフォントサイズを計算
		const size = Math.min(Math.sqrt(area / d.data.name.length) * 3, 24);
		return `${Math.max(12, size)}px`;
	}
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	let container: HTMLElement;

	// サンプルデータ - ICU患者の状態データ
	const data = {
		name: '患者状態',
		value: 100,
		children: [
			{
				name: 'バイタルサイン',
				value: 30,
				children: [
					{ name: '体温 38.5℃', value: 10, status: 'warning' },
					{ name: '血圧 125/85', value: 8, status: 'normal' },
					{ name: '心拍数 95/分', value: 7, status: 'warning' },
					{ name: 'SpO2 98%', value: 5, status: 'normal' }
				]
			},
			{
				name: '検査値',
				value: 40,
				children: [
					{ name: 'WBC 12000', value: 12, status: 'warning' },
					{ name: 'CRP 5.8', value: 10, status: 'warning' },
					{ name: 'Lactate 2.1', value: 8, status: 'warning' },
					{ name: 'BNP 150', value: 10, status: 'normal' }
				]
			},
			{
				name: '治療状況',
				value: 30,
				children: [
					{ name: '人工呼吸器', value: 15, status: 'normal' },
					{ name: '昇圧剤', value: 8, status: 'warning' },
					{ name: '鎮静剤', value: 7, status: 'normal' }
				]
			}
		]
	};

	// データの型定義を追加
	type TreeData = {
		name: string;
		value: number;
		status?: string;
		children?: TreeData[];
	};

	onMount(() => {
		const width = container.clientWidth;
		const height = container.clientHeight;

		const treemap = d3.treemap<TreeData>().size([width, height]).padding(1).round(true);

		// ソート関数を修正
		const root = d3
			.hierarchy<TreeData>(data)
			.sum((d) => d.value)
			.sort((a, b) => (b.value ?? 0) - (a.value ?? 0)) as d3.HierarchyRectangularNode<TreeData>;

		treemap(root);

		const svg = d3.select(container).append('svg').attr('width', width).attr('height', height);

		const cell = svg
			.selectAll('g')
			.data<d3.HierarchyRectangularNode<TreeData>>(root.leaves())
			.join('g')
			.attr('transform', (d) => `translate(${d.x0},${d.y0})`);

		const colorScale = d3
			.scaleOrdinal<string, string>()
			.domain(['normal', 'warning'])
			.range(['#4CAF50', '#FFA726']);

		cell
			.append('rect')
			.attr('width', (d) => d.x1 - d.x0)
			.attr('height', (d) => d.y1 - d.y0)
			.attr('fill', (d) => colorScale(d.data.status ?? 'normal'))
			.attr('opacity', 0.8);

		cell
			.append('text')
			.attr('x', 4)
			.attr('y', 14)
			.text((d) => d.data.name)
			.attr('font-size', '12px')
			.attr('fill', 'white');
	});
</script>

<div class="treemap-container" bind:this={container}></div>

<style>
	.treemap-container {
		width: 100%;
		height: 100%;
		background-color: #2c3e50;
	}
</style>
