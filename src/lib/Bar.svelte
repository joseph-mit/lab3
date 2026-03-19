<script>
  import * as d3 from 'd3';

  export let data = [];

  let width = 600;
  let height = 400;
  let margin = { top: 40, right: 150, bottom: 70, left: 60 };

  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis, yAxis;

  $: xScale = d3.scaleBand()
      .domain(data.map(d => d.label))
      .range([0, innerWidth])
      .padding(0.2);

  $: yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.value) || 1])
      .range([innerHeight, 0]);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
      .domain(data.map(d => d.label));

  $: maxBar = d3.greatest(data, d => d.value);

  $: if (xAxis && yAxis) {
      d3.select(xAxis).call(d3.axisBottom(xScale));
      d3.select(yAxis).call(
          d3.axisLeft(yScale)
            .tickFormat(d => Number.isInteger(d) ? d : "")
            .tickValues(d3.range(0, d3.max(data, d => d.value) + 1))
      );
  }
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">
    <!-- Chart title -->
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title">
      Projects per Year
    </text>

    <!-- x-axis -->
    <g transform="translate({margin.left}, {margin.top + innerHeight})"
       bind:this={xAxis} />

    <!-- y-axis -->
    <g transform="translate({margin.left}, {margin.top})"
       bind:this={yAxis} />

    <!-- Bars and labels -->
    <g transform="translate({margin.left}, {margin.top})">

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
        y={-margin.left + 15}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        Number of Projects
      </text>

      {#each data as d}
        <rect
          x={xScale(d.label)}
          y={yScale(d.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(d.value)}
          fill={colorScale(d.label)}
        />
      {/each}

      {#if maxBar}
        <!-- Highlight outline around the tallest bar -->
        <rect
          x={xScale(maxBar.label)}
          y={yScale(maxBar.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(maxBar.value)}
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        />
        <!-- Leader line -->
        <line
          x1={xScale(maxBar.label) + xScale.bandwidth()}
          y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
          x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
          y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
          stroke="currentColor"
          stroke-width="1"
        />
        <!-- Annotation text -->
        <text
          x={xScale(maxBar.label) + xScale.bandwidth() + 35}
          y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
          dominant-baseline="middle"
          class="annotation">
          Year with most projects
        </text>
      {/if}
    </g>
  </svg>

  <ul class="legend">
    {#each data as d}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch"></span>
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .container {
    display: flex;
    align-items: start;
    gap: 1rem;
  }

  .chart-title {
    font-size: 1em;
    font-weight: bold;
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.7em;
    fill: black;
    font-style: italic;
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    flex: 1;
    display: grid;
    gap: 0.6rem;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    font-size: 0.85rem;
  }

  .swatch {
    display: inline-block;
    width: 14px;
    height: 14px;
    border-radius: 2px;
    background-color: var(--color);
    flex-shrink: 0;
  }
</style>
