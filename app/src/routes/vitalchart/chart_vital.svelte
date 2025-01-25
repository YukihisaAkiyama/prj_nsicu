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
	//let height = 200 - margin.top - margin.bottom;

	let chartContainer: HTMLDivElement;
	const margin = { top: 20, right: 50, bottom: 30, left: 50 };
	const width = 1000 - margin.left - margin.right;
	const height = 230 - margin.top - margin.bottom;

	onMount(() => {
		const data = generateSampleData();

		// ツールチップの作成
		const tooltip = d3
			.select(chartContainer)
			.append('div')
			.style('position', 'absolute')
			.style('visibility', 'hidden')
			.style('background-color', 'white')
			.style('border', '1px solid #ddd')
			.style('padding', '10px')
			.style('border-radius', '5px')
			.style('box-shadow', '2px 2px 6px rgba(0, 0, 0, 0.3)');

		// SVG作成
		const svg = d3
			.select(chartContainer)
			.append('svg')
			.attr('width', width + margin.left + margin.right)
			.attr('height', height + margin.top + margin.bottom)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);

		// スケール設定
		const x = d3
			.scaleTime()
			.domain(d3.extent(data, (d) => d.timestamp) as [Date, Date])
			.range([0, width]);

		const y1 = d3
			.scaleLinear() // 体温用
			.domain([35, 38])
			.range([height, 0]);

		const y2 = d3
			.scaleLinear() // 血圧用
			.domain([40, 160])
			.range([height, 0]);

		const y3 = d3
			.scaleLinear() // SpO2用
			.domain([90, 100])
			.range([height, 0]);

		// ライン生成関数
		const tempLine = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y1(d.temperature));

		const systolicLine = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y2(d.systolic));

		const diastolicLine = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y2(d.diastolic));

		const pulseLine = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y2(d.pulse));

		const spo2Line = d3
			.line<any>()
			.x((d) => x(d.timestamp))
			.y((d) => y3(d.spo2));

		// X軸
		svg.append('g').attr('transform', `translate(0,${height})`).call(d3.axisBottom(x));

		// Y軸（左：体温）
		svg
			.append('g')
			.call(d3.axisLeft(y1))
			.append('text')
			.attr('fill', '#000')
			.attr('y', -10)
			.text('体温 (℃)');

		// Y軸（中央：血圧・脈拍）
		svg
			.append('g')
			.attr('transform', `translate(${width},0)`)
			.call(d3.axisRight(y2))
			.append('text')
			.attr('fill', '#000')
			.attr('y', -10)
			.text('血圧 (mmHg) / 脈拍 (bpm)');

		// Y軸（右：SpO2）
		svg
			.append('g')
			.attr('transform', `translate(${width + 40},0)`)
			.call(d3.axisRight(y3))
			.append('text')
			.attr('fill', '#000')
			.attr('y', -10)
			.text('SpO2 (%)');

		// データポイントとツールチップの追加
		const formatTime = d3.timeFormat('%Y/%m/%d %H:%M');

		// 各データポイントのグループを作成
		const points = svg.append('g').attr('class', 'points');

		// 体温のポイント
		points
			.selectAll('.temp-point')
			.data(data)
			.enter()
			.append('circle')
			.attr('class', 'temp-point')
			.attr('cx', (d) => x(d.timestamp))
			.attr('cy', (d) => y1(d.temperature))
			.attr('r', 3)
			.attr('fill', 'red')
			.on('mouseover', (event, d) => {
				tooltip
					.style('visibility', 'visible')
					.html(`時刻: ${formatTime(d.timestamp)}<br>体温: ${d.temperature.toFixed(1)}℃`)
					.style('left', `${event.pageX + 10}px`)
					.style('top', `${event.pageY - 10}px`);
			})
			.on('mouseout', () => {
				tooltip.style('visibility', 'hidden');
			});

		// 血圧のポイント
		points
			.selectAll('.bp-point')
			.data(data)
			.enter()
			.append('circle')
			.attr('class', 'bp-point')
			.attr('cx', (d) => x(d.timestamp))
			.attr('cy', (d) => y2(d.systolic))
			.attr('r', 3)
			.attr('fill', 'blue')
			.on('mouseover', (event, d) => {
				tooltip
					.style('visibility', 'visible')
					.html(
						`時刻: ${formatTime(d.timestamp)}<br>収縮期血圧: ${Math.round(
							d.systolic
						)}mmHg<br>拡張期血圧: ${Math.round(d.diastolic)}mmHg`
					)
					.style('left', `${event.pageX + 10}px`)
					.style('top', `${event.pageY - 10}px`);
			})
			.on('mouseout', () => {
				tooltip.style('visibility', 'hidden');
			});

		points
			.selectAll('.dbp-point')
			.data(data)
			.enter()
			.append('circle')
			.attr('class', 'dbp-point')
			.attr('cx', (d) => x(d.timestamp))
			.attr('cy', (d) => y2(d.diastolic))
			.attr('r', 3)
			.attr('fill', 'lightblue');

		// 脈拍のポイント
		points
			.selectAll('.pulse-point')
			.data(data)
			.enter()
			.append('circle')
			.attr('class', 'pulse-point')
			.attr('cx', (d) => x(d.timestamp))
			.attr('cy', (d) => y2(d.pulse))
			.attr('r', 3)
			.attr('fill', 'green')
			.on('mouseover', (event, d) => {
				tooltip
					.style('visibility', 'visible')
					.html(`時刻: ${formatTime(d.timestamp)}<br>脈拍: ${Math.round(d.pulse)}bpm`)
					.style('left', `${event.pageX + 10}px`)
					.style('top', `${event.pageY - 10}px`);
			})
			.on('mouseout', () => {
				tooltip.style('visibility', 'hidden');
			});

		// SpO2のポイント
		points
			.selectAll('.spo2-point')
			.data(data)
			.enter()
			.append('circle')
			.attr('class', 'spo2-point')
			.attr('cx', (d) => x(d.timestamp))
			.attr('cy', (d) => y3(d.spo2))
			.attr('r', 3)
			.attr('fill', 'purple')
			.on('mouseover', (event, d) => {
				tooltip
					.style('visibility', 'visible')
					.html(`時刻: ${formatTime(d.timestamp)}<br>SpO2: ${Math.round(d.spo2)}%`)
					.style('left', `${event.pageX + 10}px`)
					.style('top', `${event.pageY - 10}px`);
			})
			.on('mouseout', () => {
				tooltip.style('visibility', 'hidden');
			});

		// ライン描画
		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', 'red')
			.attr('stroke-width', 1.5)
			.attr('d', tempLine);

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', 'blue')
			.attr('stroke-width', 1.5)
			.attr('d', systolicLine);

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', 'lightblue')
			.attr('stroke-width', 1.5)
			.attr('d', diastolicLine);

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', 'green')
			.attr('stroke-width', 1.5)
			.attr('d', pulseLine);

		svg
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', 'purple')
			.attr('stroke-width', 1.5)
			.attr('d', spo2Line);

		// 凡例
		const legend = svg
			.append('g')
			.attr('font-family', 'sans-serif')
			.attr('font-size', 10)
			.attr('text-anchor', 'start')
			.selectAll('g')
			.data(['体温', '収縮期血圧', '拡張期血圧', '脈拍', 'SpO2'])
			.enter()
			.append('g')
			.attr('transform', (d, i) => `translate(0,${i * 20})`);

		legend
			.append('rect')
			.attr('x', width - 150)
			.attr('width', 19)
			.attr('height', 19)
			.attr('fill', (d, i) => ['red', 'blue', 'lightblue', 'green', 'purple'][i]);

		legend
			.append('text')
			.attr('x', width - 125)
			.attr('y', 9.5)
			.attr('dy', '0.32em')
			.text((d) => d);
	});
</script>

<div bind:this={chartContainer} class="h-full w-full"></div>

<style>
	div {
		background-color: white;
		padding: 1rem;
	}
</style>
