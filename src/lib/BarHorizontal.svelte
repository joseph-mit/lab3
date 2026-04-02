<script>
  import * as d3 from 'd3';

  export let data = [];
  export let title = '';

  let width = 650;
  let height = 220;
  let margin = { top: 40, right: 110, bottom: 50, left: 75 };

  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  let xAxis, yAxis;

  $: xScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value) || 1])
    .range([0, innerWidth])
    .nice();

  $: yScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerHeight])
    .padding(0.25);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

  $: maxBar = d3.greatest(data, d => d.value);

  $: if (xAxis && yAxis) {
    d3.select(xAxis)
      .call(
        d3.axisBottom(xScale)
          .ticks(Math.min(d3.max(data, d => d.value) || 10, 10))
          .tickFormat(d => Number.isInteger(d) ? d : '')
      )
      .call(g => g.select('.domain').remove())
      .call(g => g.selectAll('.tick line').remove());

    d3.select(yAxis)
      .call(d3.axisLeft(yScale))
      .call(g => g.select('.domain').remove())
      .call(g => g.selectAll('.tick line').remove());
  }
</script>

<div class="chart-wrapper">
  <svg viewBox="0 0 {width} {height}">
    <!-- Chart title -->
    <text
      x={margin.left + innerWidth / 2}
      y={20}
      text-anchor="middle"
      class="chart-title">
      {title || 'Lines of Code by Language'}
    </text>

    <!-- Axes -->
    <g transform="translate({margin.left}, {margin.top + innerHeight})"
       bind:this={xAxis} class="axis x-axis" />

    <g transform="translate({margin.left}, {margin.top})"
       bind:this={yAxis} class="axis y-axis" />

    <!-- Chart area -->
    <g transform="translate({margin.left}, {margin.top})">

      <!-- Subtle vertical grid lines -->
      {#each xScale.ticks() as tick}
        <line
          x1={xScale(tick)}
          y1={0}
          x2={xScale(tick)}
          y2={innerHeight}
          stroke="#e0e0e0"
          stroke-dasharray="3,3"
        />
      {/each}

      <!-- Bars -->
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
          rx="3"
          ry="3"
        />
      {/each}

      <!-- Annotation -->
      {#if maxBar && maxBar.value > 0}
        <rect
          x={-1.5}
          y={yScale(maxBar.label) - 1.5}
          width={xScale(maxBar.value) + 3}
          height={yScale.bandwidth() + 3}
          fill="none"
          stroke="#333"
          stroke-width="2"
          rx="3"
          ry="3"
        />
        <text
          x={xScale(maxBar.value) + 6}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="middle"
          text-anchor="start"
          class="annotation">
          Most lines of code
        </text>
      {/if}

      <!-- x-axis label -->
      <text
        x={innerWidth / 2}
        y={innerHeight + 40}
        text-anchor="middle"
        class="axis-label">
        Lines of Code
      </text>

      <!-- y-axis label -->
      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 12}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        <tspan x={-(innerHeight / 2)} dy="0">Programming</tspan>
        <tspan x={-(innerHeight / 2)} dy="1.1em">Language</tspan>
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
    margin: 1rem 0 2rem;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
    flex: 0 1 650px;
  }

  .chart-title {
    font-size: 0.95em;
    font-weight: 700;
    fill: currentColor;
    letter-spacing: -0.01em;
  }

  .axis-label {
    font-size: 0.68em;
    fill: #555;
    font-weight: 500;
  }

  .annotation {
    font-size: 0.6em;
    fill: #444;
    font-style: italic;
    font-weight: 500;
  }

  :global(.axis text) {
    font-size: 10px;
    fill: #555;
  }

  .legend {
    list-style: none;
    padding: 0.5rem 0;
    margin: 0;
    display: grid;
    gap: 0.35rem;
    align-self: center;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    font-size: 0.75rem;
    line-height: 1.2;
  }

  .swatch {
    display: inline-block;
    width: 11px;
    height: 11px;
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
