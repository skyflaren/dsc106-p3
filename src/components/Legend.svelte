<script>
	// receive color scale as prop.
	export let colorScale;

	// determine thresholds to display.
	let thresholds = colorScale.thresholds();

	// add 0 to thresholds to show color
	// that encodes the lowest bucket.
	thresholds.unshift(0);
</script>

<g class="legend">
		<!-- Add the title of the legend. -->
		<text 
			class="legend-title" 
			fill="currentColor"
			font-weight="bold"
			x="610" y="20"
			>
			Unemployment rate (%)
		</text>
	
		<!-- Loop through each of the thresholds. -->
		{#each thresholds as tick, i}
			{@const xPosition = 610 + (i * 30)}
			{@const yPosition = 30}

			<!-- Add a square for each threshold in its respective color. -->
			<rect
				fill={colorScale(tick)}
				x={xPosition}
				y={yPosition}
				height="10"
				width="30"
			/>

			<!-- Skip the first threshold, but for the rest... -->
			{#if i !== 0}

				<!-- ...add a vertical tick line, -->
				<line
	        stroke="currentColor"
	        x1={xPosition}
	        x2={xPosition}
	        y1={yPosition}
	        y2={yPosition + 20}
	      />

				<!-- ...and a tick label. -->
				<text
	        fill="currentColor"
	        text-anchor="middle"
	        dominant-baseline="middle"
	        x={xPosition}
	        y={yPosition + 30}
	      >
	        {tick}
	      </text>
			{/if}
		{/each}
</g>
