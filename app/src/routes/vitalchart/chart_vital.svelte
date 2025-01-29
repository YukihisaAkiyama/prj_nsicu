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
		console.log('data', data);
		return data;
	};
	//let height = 200 - margin.top - margin.bottom;

	// 色の定義
	const colors = {
		temperature: '#00f', // 青
		systolic: '#050', // 緑
		diastolic: '#080', // 緑
		pulse: '#f00', //
		spo2: '#08f' // 黒（反転の場合は白）
	};

	let chartContainer: HTMLDivElement;
	//const margin = { top: 20, right: 50, bottom: 30, left: 50 };
	const margin = { top: 20, right: 50, bottom: 30, left: 200 };
	const width = 1200 - margin.left - margin.right;
	const height = 240 - margin.top - margin.bottom;

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

		// クリップパスを定義
		svg
			.append('defs')
			.append('clipPath')
			.attr('id', 'clip')
			.append('rect')
			.attr('width', width)
			.attr('height', height);

		// スケール設定
		const x = d3
			.scaleTime()
			.domain(d3.extent(data, (d) => d.timestamp) as [Date, Date])
			.range([0, width]);

		const y1 = d3
			.scaleLinear() // 体温用
			.domain([35, 40])
			.range([height, 0]);

		const y2 = d3
			.scaleLinear() // 血圧用
			.domain([40, 200])
			.range([height, 0]);

		const y3 = d3
			.scaleLinear() // SpO2用
			.domain([70, 100])
			.range([height, 0]);

		// ズーム機能の追加
		const zoom = d3
			.zoom()
			.scaleExtent([1, 10])
			.extent([
				[0, 0],
				[width, height]
			])
			.on('zoom', zoomed);

		// ズーム用の透明な四角形を追加
		svg
			.append('rect')
			.attr('class', 'zoom')
			.attr('width', width)
			.attr('height', height)
			.style('fill', 'none')
			.style('pointer-events', 'all');

		// グラフ要素をグループ化
		const chartGroup = svg.append('g').attr('clip-path', 'url(#clip)');

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
		// X軸の目盛りを1日ごとに設定
		// X軸の目盛り設定

		const xAxisTicksHourly = x.ticks(d3.timeHour); // 1時間ごとの目盛り
		const xAxisTicksDaily = x.ticks(d3.timeDay); // 1日ごとの目盛り
		const xAxisFormatHourly = d3.timeFormat('%H:%M'); // 時:分表示
		const xAxisFormatDaily = d3.timeFormat('%m/%d'); // 月/日表示

		// ズームレベルに応じて目盛りの間隔を変更する関数
		function updateXAxisTicks(scale: any) {
			const domain = scale.domain();
			const hours = (domain[1].getTime() - domain[0].getTime()) / (1000 * 60 * 60);

			if (hours <= 24) {
				// 表示期間が24時間以内の場合
				return d3
					.axisBottom(scale)
					.tickValues(xAxisTicksHourly)
					.tickFormat((d: any) => xAxisFormatHourly(d));
			} else {
				return d3
					.axisBottom(scale)
					.tickValues(xAxisTicksDaily)
					.tickFormat((d: any) => xAxisFormatDaily(d));
			}
		}
		const xAxisTicks = x.ticks(d3.timeDay);
		const xAxisFormat = d3.timeFormat('%m/%d');
		// X軸グリッドライン
		const xAxisGrid = svg
			.append('g')
			.attr('class', 'grid')
			.attr('transform', `translate(0,${height})`)
			.call(
				d3
					.axisBottom(x)
					.tickSize(-height)
					.tickFormat(() => '')
			)
			.call((g) => g.select('.domain').remove())
			.attr('opacity', 0.1);
		const xAxis = svg
			.append('g')
			.attr('transform', `translate(0,${height})`)
			.call(d3.axisBottom(x));

		// y軸グリッドライン：HR:BP(y2)に付ける事で解決する
		svg
			.append('g')
			.attr('class', 'grid')
			.attr('transform', `translate(${0},0)`)
			.call(
				d3
					.axisLeft(y2)
					.tickSize(-width)
					.tickFormat(() => '')
			)
			.call((g) => g.select('.domain').remove())
			.attr('opacity', 0.1);
		// Y軸：血圧 (mmHg) / 脈拍 (bpm)
		svg
			.append('g')
			.attr('transform', `translate(${0},0)`)
			.call(d3.axisLeft(y2))
			.attr('color', colors.diastolic)
			.append('text')
			.attr('fill', colors.diastolic)
			.attr('y', -10)
			.text('HR/BP');

		// Y軸：SpO2 (%)

		svg
			.append('g')
			.attr('transform', `translate(${-60},0)`)
			.call(d3.axisLeft(y3))
			.attr('color', colors.spo2)
			.append('text')
			.attr('fill', colors.spo2)
			.attr('y', -10)
			.text('SpO2');

		// Y軸:体温 (℃)
		svg
			.append('g')
			.attr('transform', `translate(${-110},0)`)
			.call(d3.axisLeft(y1))
			.attr('color', colors.temperature)
			.append('text')
			.attr('fill', colors.temperature)
			.attr('y', -10)
			.text('BT');

		// Y軸（右：脈拍）
		svg
			.append('g')
			.attr('transform', `translate(${-170},0)`)
			.call(d3.axisLeft(y2))
			.attr('color', colors.pulse)
			.append('text')
			.attr('fill', colors.pulse)
			.attr('y', -10)
			.text('Pulse');
		// データポイントとツールチップの追加
		const formatTime = d3.timeFormat('%Y/%m/%d %H:%M');

		// 各データポイントのグループを作成
		const points = chartGroup.append('g').attr('class', 'points');

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
			.attr('fill', colors.temperature)
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
			.append('path')
			.attr('d', d3.symbol().type(d3.symbolTriangle2).size(50))
			.attr('transform', (d) => `translate(${x(d.timestamp)},${y2(d.systolic)}) rotate(180)`)
			.attr('class', 'bp-point')
			.attr('cx', (d) => x(d.timestamp))
			.attr('cy', (d) => y2(d.systolic))
			.attr('r', 3)
			.attr('fill', colors.systolic)
			.on('mouseover', (event, d) => {
				tooltip
					.style('visibility', 'visible')
					.html(
						`時刻: ${formatTime(d.timestamp)}<br>収縮期血圧高: ${Math.round(
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
			.append('path')
			.attr('d', d3.symbol().type(d3.symbolTriangle2).size(50))
			.attr('transform', (d) => `translate(${x(d.timestamp)},${y2(d.diastolic)})`)
			.attr('class', 'dbp-point')
			.attr('fill', colors.diastolic)
			.on('mouseover', (event, d) => {
				tooltip
					.style('visibility', 'visible')
					.html(
						`時刻: ${formatTime(d.timestamp)}<br>収縮期血圧高: ${Math.round(
							d.systolic
						)}mmHg<br>拡張期血圧: ${Math.round(d.diastolic)}mmHg`
					)
					.style('left', `${event.pageX + 10}px`)
					.style('top', `${event.pageY - 10}px`);
			})
			.on('mouseout', () => {
				tooltip.style('visibility', 'hidden');
			});

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
			.attr('fill', colors.pulse)
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
			.attr('fill', colors.spo2)
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
		chartGroup
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.temperature)
			.attr('stroke-width', 1.5)
			.attr('d', tempLine);

		chartGroup
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.systolic)
			.attr('stroke-width', 1.5)
			.attr('d', systolicLine);

		chartGroup
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.diastolic)
			.attr('stroke-width', 1.5)
			.attr('d', diastolicLine);

		chartGroup
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.pulse)
			.attr('stroke-width', 1.5)
			.attr('d', pulseLine);

		chartGroup
			.append('path')
			.datum(data)
			.attr('fill', 'none')
			.attr('stroke', colors.spo2)
			.attr('stroke-width', 1.5)
			.attr('d', spo2Line);

		// 凡例
		/*
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
			*/
		// ズーム機能を適用
		svg.call(zoom as any);

		// ズーム時の更新関数
		function zoomed(event: any) {
			const newX = event.transform.rescaleX(x);

			// X軸の更新
			xAxis.call(d3.axisBottom(newX));

			// グリッドラインの更新
			xAxisGrid
				.call(
					d3
						.axisBottom(newX)
						.tickSize(-height)
						.tickFormat(() => '')
				)
				.call((g) => g.select('.domain').remove());

			// ラインの更新
			chartGroup.selectAll('path').attr('d', function (this: any) {
				const line = d3.select(this);
				line.attr('fill', 'none');
				if (line.attr('stroke') === colors.temperature)
					return tempLine.x((d: any) => newX(d.timestamp))(data);
				if (line.attr('stroke') === colors.systolic)
					return systolicLine.x((d: any) => newX(d.timestamp))(data);
				if (line.attr('stroke') === colors.diastolic)
					return diastolicLine.x((d: any) => newX(d.timestamp))(data);
				if (line.attr('stroke') === colors.pulse)
					return pulseLine.x((d: any) => newX(d.timestamp))(data);
				if (line.attr('stroke') === colors.spo2)
					return spo2Line.x((d: any) => newX(d.timestamp))(data);
				return spo2Line.x((d: any) => newX(d.timestamp))(data);
			});

			// ポイントの更新
			points.selectAll('circle').attr('cx', (d: any) => newX(d.timestamp));
		}
	});
</script>

<div bind:this={chartContainer} class="h-full w-full"></div>

<style>
	div {
		background-color: white;
		padding: 1rem;
	}
</style>
