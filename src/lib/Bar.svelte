<script>
  import * as d3 from 'd3';

  export let data = [];

  let width = 650;
  let height = 420;
  let margin = { top: 50, right: 160, bottom: 75, left: 70 };

  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  let xAxis, yAxis;

  $: xScale = d3.scaleBand()
      .domain(data.map(d => d.label))
      .range([0, innerWidth])
      .padding(0.25);

  $: yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.value) || 1])
      .range([innerHeight, 0]);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
      .domain(data.map(d => d.label));

  $: maxBar = d3.greatest(data, d => d.value);

  $: yTicks = d3.range(0, (d3.max(data, d => d.value) || 1) + 1);

  $: if (xAxis && yAxis) {
      d3.select(xAxis)
        .call(d3.axisBottom(xScale))
        .call(g => g.select('.domain').remove())
        .call(g => g.selectAll('.tick line').remove());

      d3.select(yAxis)
        .call(
          d3.axisLeft(yScale)
            .tickFormat(d => Number.isInteger(d) ? d : '')
            .tickValues(yTicks)
            .tickSize(-innerWidth)
        )
        .call(g => g.select('.domain').remove())
        .call(g => g.selectAll('.tick line')
          .attr('stroke', '#e0e0e0')
          .attr('stroke-dasharray', '3,3'));
  }
</script>

<div class="chart-wrapper">
  <svg viewBox="0 0 {width} {height}">
    <!-- Chart title -->
    <text
      x={margin.left + innerWidth / 2}
      y={24}
      text-anchor="middle"
      class="chart-title">
      Projects per Year
    </text>

    <!-- y-axis (rendered first so grid lines are behind bars) -->
    <g transform="translate({margin.left}, {margin.top})"
       bind:this={yAxis} class="axis y-axis" />

    <!-- x-axis -->
    <g transform="translate({margin.left}, {margin.top + innerHeight})"
       bind:this={xAxis} class="axis x-axis" />

    <!-- Chart area -->
    <g transform="translate({margin.left}, {margin.top})">

      <!-- Bars -->
      {#each data as d}
        <rect
          x={xScale(d.label)}
          y={yScale(d.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(d.value)}
          fill={colorScale(d.label)}
          rx="3"
          ry="3"
        />
      {/each}

      <!-- Annotation -->
      {#if maxBar}
        <rect
          x={xScale(maxBar.label) - 1.5}
          y={yScale(maxBar.value) - 1.5}
          width={xScale.bandwidth() + 3}
          height={innerHeight - yScale(maxBar.value) + 1.5}
          fill="none"
          stroke="#333"
          stroke-width="2"
          rx="3"
          ry="3"
        />
        <line
          x1={xScale(maxBar.label) + xScale.bandwidth() + 4}
          y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
          x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
          y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
          stroke="#555"
          stroke-width="1.2"
        />
        <text
          x={xScale(maxBar.label) + xScale.bandwidth() + 34}
          y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
          dominant-baseline="middle"
          class="annotation">
          Year with most projects
        </text>
      {/if}

      <!-- x-axis label -->
      <text
        x={innerWidth / 2}
        y={innerHeight + 50}
        text-anchor="middle"
        class="axis-label">
        Year
      </text>

      <!-- y-axis label -->
      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 18}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        Number of Projects
      </text>
    </g>
  </svg>

  <ul class="legend">
    {#each data as d}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch"></span>
        <span class="legend-label">{d.label}</span>
        <span class="legend-value">({d.value})</span>
      </li>
    {/each}
  </ul>
</div>

<style>
  .chart-wrapper {
    display: flex;
    align-items: start;
    gap: 0.75rem;
    margin: 1.5rem 0 2rem;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
    flex: 0 1 650px;
  }

  .chart-title {
    font-size: 1.15em;
    font-weight: 700;
    fill: currentColor;
    letter-spacing: -0.01em;
  }

  .axis-label {
    font-size: 0.78em;
    fill: #555;
    font-weight: 500;
  }

  .annotation {
    font-size: 0.68em;
    fill: #444;
    font-style: italic;
    font-weight: 500;
  }

  :global(.axis text) {
    font-size: 11px;
    fill: #555;
  }

  .legend {
    list-style: none;
    padding: 0.5rem 0;
    margin: 0;
    display: grid;
    gap: 0.45rem;
    align-self: center;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.45rem;
    font-size: 0.82rem;
    line-height: 1.2;
  }

  .swatch {
    display: inline-block;
    width: 13px;
    height: 13px;
    border-radius: 3px;
    background-color: var(--color);
    flex-shrink: 0;
  }

  .legend-label {
    font-weight: 500;
    color: #333;
  }

  .legend-value {
    color: #777;
    font-size: 0.9em;
  }
</style>
