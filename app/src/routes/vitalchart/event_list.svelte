<script lang="ts">
	// カテゴリのサンプルデータ
	const categories = [
		'バイタル',
		'投薬',
		'処置',
		'食事',
		'排泄',
		'清潔',
		'移動',
		'睡眠',
		'リハビリ',
		'面会',
		'検査',
		'手術',
		'診察',
		'相談',
		'指導',
		'転倒',
		'感染',
		'アレルギー',
		'その他',
		'記録'
	];
</script>

<button
	class="button absolute right-1 m-8 rounded-md border border-e-2 px-4 py-2 text-white shadow-md"
>
	button
</button>

<div class="">
	{#each categories as category, categoryIndex}
		<table class="mb-2 w-full border-collapse border border-gray-300">
			<tbody>
				{#each Array(5) as _, rowIndex}
					<tr>
						{#each Array(28) as _, colIndex}
							<td
								class="border border-gray-300 p-1 text-sm {colIndex % 2 === 0
									? 'bg-gray-50'
									: 'bg-white'}"
							>
								{#if colIndex === 0 && rowIndex === 0}
									<span class="font-bold">{category}</span>
								{/if}
							</td>
						{/each}
					</tr>
				{/each}
			</tbody>
		</table>
	{/each}
</div>

<style>
	td {
		width: 60px;
		min-width: 60px;
		max-width: 60px;
	}

	.button {
		@apply bg-gradient-to-br from-violet-800 via-pink-700 to-orange-700;
	}
</style>
