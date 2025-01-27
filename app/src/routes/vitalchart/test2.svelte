<script lang="ts">
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	// サンプルデータの生成 (14日分、1日3回)
	const generateSampleData = () => {
		const data = [];
		const startDate = new Date(2024, 0, 1);

		for (let day = 0; day < 14; day++) {
			for (let time = 0; time < 3; time++) {
				const date = new Date(startDate);
				date.setDate(startDate.getDate() + day);
				date.setHours(6 + time * 6); // 6時、12時、18時

				data.push({
					timestamp: date,
					temperature: 36.2 + Math.random() * 1.5, // 36.2-37.7℃
					systolic: 110 + Math.random() * 30, // 110-140 mmHg
					diastolic: 60 + Math.random() * 20, // 60-80 mmHg
					pulse: 65 + Math.random() * 25, // 65-90 bpm
					spo2: 95 + Math.random() * 4 // 95-99%
				});
			}
		}
		return data;
	};

	// 色の定義
	const colors = {
		temperature: '#00f', // 青
		systolic: '#050', // 緑
		diastolic: '#080', // 緑
		pulse: '#f00', // 赤
		spo2: '#888' // グレー
	};

	let chartContainer: HTMLDivElement;
	const margin = { top: 20, right: 50, bottom: 30, left: 50 };
	const width = 1000 - margin.left - margin.right;
	const height = 500 - margin.top - margin.bottom;

	onMount(() => {
		const data = generateSampleData();

		// SVG作成
		const svg = d3
			.select(chartContainer)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);

		// X軸のスケール設定
		const x = d3
			.scaleTime()
			.domain(d3.extent(data, (d) => d.timestamp) as [Date, Date])
			.range([0, width]);

		// Y軸のスケール設定
		const y1 = d3.scaleLinear().domain([35, 42]).range([height, 0]); // 体温
		const y2 = d3.scaleLinear().domain([0, 200]).range([height, 0]); // 血圧
		const y3 = d3.scaleLinear().domain([40, 120]).range([height, 0]); // 脈拍
		const y4 = d3.scaleLinear().domain([80, 100]).range([height, 0]); // SpO2

		// X軸の描画
		svg.append('g').attr('transform', `translate(0,${height})`).call(d3.axisBottom(x));

		// Y軸の描画（左側：体温）
		svg.append('g').call(d3.axisLeft(y1));

		// Y軸の描画（右側：血圧、脈拍、SpO2）
		svg.append('g').attr('transform', `translate(${width},0)`).call(d3.axisRight(y2));

		// ライン生成関数
		const line = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y1(d.temperature));

		// 体温のライン描画
		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.temperature)
			.attr('stroke-width', 1.5)
			.attr('d', line);

		// 血圧のライン描画（収縮期・拡張期）
		const bpLine = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y2(d.systolic));

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.systolic)
			.attr('stroke-width', 1.5)
			.attr('d', bpLine);

		const bpLine2 = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y2(d.diastolic));

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.diastolic)
			.attr('stroke-width', 1.5)
			.attr('d', bpLine2);

		// 脈拍のライン描画
		const pulseLine = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y3(d.pulse));

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.pulse)
			.attr('stroke-width', 1.5)
			.attr('d', pulseLine);

		// SpO2のライン描画
		const spo2Line = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y4(d.spo2));

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.spo2)
			.attr('stroke-width', 1.5)
			.attr('d', spo2Line);
	});
</script>

<div bind:this={chartContainer} class="h-full w-full"></div>

<style>
	div {
		background-color: white;
		padding: 1rem;
	}
</style>
