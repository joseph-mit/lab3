<script>
  import * as d3 from 'd3';

  export let data = [];

  let width = 1000;
  let height = 300;
  let margin = { top: 20, right: 20, bottom: 45, left: 60 };

  $: usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
  };

  $: xScale = d3.scaleTime()
    .domain(d3.extent(data, d => d.date) ?? [new Date(), new Date()])
    .range([usableArea.left, usableArea.right]);

  $: yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.count) ?? 1])
    .range([usableArea.bottom, usableArea.top])
    .nice();

  $: line = d3.line()
    .x(d => xScale(d.date))
    .y(d => yScale(d.count))
    .curve(d3.curveBumpX);

  let xAxisEl, yAxisEl;

  $: if (xAxisEl) {
    d3.select(xAxisEl).call(d3.axisBottom(xScale));
  }

  $: if (yAxisEl) {
    d3.select(yAxisEl).call(d3.axisLeft(yScale));
  }

  // Day-of-week highlighting
  let hoveredDay = null;

  $: dayRegions = (() => {
    if (data.length === 0) return [];
    return data.map((d, i, arr) => {
      const prev = arr[i - 1];
      const next = arr[i + 1];
      const left = prev ? new Date((d.date.getTime() + prev.date.getTime()) / 2) : d.date;
      const right = next ? new Date((d.date.getTime() + next.date.getTime()) / 2) : d.date;
      return {
        date: d.date,
        weekday: d.date.toLocaleString("en", { weekday: "long" }),
        x: xScale(left),
        width: xScale(right) - xScale(left),
      };
    });
  })();
</script>

<h3 class="chart-heading">
  {hoveredDay ? `Lines Edited on ${hoveredDay}s` : 'Lines Edited by Day'}
</h3>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<svg viewBox="0 0 {width} {height}" on:mouseleave={() => hoveredDay = null}>
  <!-- Highlight bands (behind everything) -->
  {#each dayRegions as region}
    {#if region.weekday === hoveredDay}
      <rect
        x={region.x}
        y={usableArea.top}
        width={region.width}
        height={usableArea.bottom - usableArea.top}
        fill="var(--color-accent)"
        opacity="0.2"
      />
    {/if}
  {/each}

  <!-- Axes -->
  <g transform="translate(0, {usableArea.bottom})" bind:this={xAxisEl} />
  <g transform="translate({usableArea.left}, 0)" bind:this={yAxisEl} />

  <!-- Axis labels -->
  <text
    x={usableArea.left + (usableArea.right - usableArea.left) / 2}
    y={height - 5}
    text-anchor="middle"
    class="axis-label">
    Date
  </text>

  <text
    x={-(usableArea.top + (usableArea.bottom - usableArea.top) / 2)}
    y={10}
    text-anchor="middle"
    transform="rotate(-90)"
    class="axis-label">
    Number of Lines Edited
  </text>

  <!-- Line -->
  <path
    d={line(data)}
    fill="none"
    stroke="steelblue"
    stroke-width="2"
  />

  <!-- Dots and annotations -->
  {#each data as d}
    {@const isHighlighted = d.date.toLocaleString("en", { weekday: "long" }) === hoveredDay}
    <circle
      cx={xScale(d.date)}
      cy={yScale(d.count)}
      r={isHighlighted ? 5 : 3}
      fill={isHighlighted ? 'var(--color-accent)' : 'steelblue'}
    />
    {#if isHighlighted}
      <text
        x={xScale(d.date)}
        y={usableArea.top + 15}
        text-anchor="middle"
        font-size="12"
        fill="var(--color-accent)"
      >
        {Math.round(d.count)}
      </text>
    {/if}
  {/each}

  <!-- Invisible hover regions (on top to catch events) -->
  {#each dayRegions as region}
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <rect
      x={region.x}
      y={usableArea.top}
      width={region.width}
      height={usableArea.bottom - usableArea.top}
      fill="transparent"
      on:mouseenter={() => hoveredDay = region.weekday}
    />
  {/each}
</svg>

<style>
  .chart-heading {
    text-align: center;
    margin: 1.5rem 0 0.5rem;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }

  :global(.axis text) {
    font-size: 11px;
    fill: #555;
  }
</style>
