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

  // Colorblind-safe sequential Blues palette, skipping the lightest stop
  // so no bar vanishes against a light canvas.
  $: colorScale = d3.scaleOrdinal()
      .domain(data.map(d => d.label))
      .range(d3.quantize(d3.interpolateBlues, data.length + 1).slice(1));

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
          .attr('stroke', 'currentColor')
          .attr('stroke-opacity', 0.18)
          .attr('stroke-dasharray', '3,3'));
  }

// ---------- Accessibility state ----------
  // We store the selected bar by its *label* (e.g. "2022"), not by its
  // array index. Labels are stable across reactive re-renders; indices
  // are not. This prevents selection from getting visually reset when
  // the parent page hands us a new data array reference.
  let selectedLabel = null;
  let liveText = '';
  let showChart = true;

  // Dynamic SVG description that stays in sync with the data.
  $: description = data.length > 0
    ? `A bar chart showing project counts by year. ${
        data
          .map(d => `${d.label}: ${d.value} ${d.value === 1 ? 'project' : 'projects'}`)
          .join(', ')
      }.`
    : 'A bar chart showing project counts by year. No data available.';

  // The opacity for a given bar, computed directly from selectedIndex.
  // -1 means nothing is selected, so everything is fully opaque.
function barOpacity(label) {
    return selectedLabel === null || selectedLabel === label ? 1 : 0.45;
  }

  function toggleBar(d, event) {
    if (!event.key || event.key === 'Enter') {
      selectedLabel = d.label;
      liveText = `${d.label}: ${d.value} ${d.value === 1 ? 'project' : 'projects'} selected.`;
    }
  }

  function toggleView() {
    showChart = !showChart;
    liveText = showChart ? 'Bar chart view shown.' : 'Table view shown.';
  }
</script>

<button
  type="button"
  class="toggle-button"
  aria-pressed={!showChart}
  aria-label="Toggle between bar chart and table view"
  on:click={toggleView}
>
  {showChart ? 'Show Table' : 'Show Chart'}
</button>

{#if showChart}
  <div class="container">
    <svg
      viewBox="0 0 {width} {height}"
      role="img"
      aria-labelledby="bar-title bar-desc"
    >
      <title id="bar-title">Projects by Year</title>
      <desc id="bar-desc">{description}</desc>

      <text
        x={margin.left + innerWidth / 2}
        y={24}
        text-anchor="middle"
        class="chart-title"
        aria-hidden="true"
      >
        Projects per Year
      </text>

      <g transform="translate({margin.left}, {margin.top})"
         bind:this={yAxis} class="axis y-axis" aria-hidden="true" />

      <g transform="translate({margin.left}, {margin.top + innerHeight})"
         bind:this={xAxis} class="axis x-axis" aria-hidden="true" />

      <g transform="translate({margin.left}, {margin.top})">

        <!-- Data bars: keyboard focusable, labeled, selectable.
             Opacity is driven directly by selectedIndex — not by CSS
             pseudo-classes — so mouse clicks and keyboard Enter both
             produce identical, deterministic dimming. -->
        {#each data as d (d.label)}
          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <rect
            class="bar"
            x={xScale(d.label)}
            y={yScale(d.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(d.value)}
            fill={colorScale(d.label)}
            stroke="currentColor"
            style:opacity={barOpacity(d.label)}
            rx="3"
            ry="3"
            tabindex="0"
            role="button"
            aria-label={`${d.label}: ${d.value} ${d.value === 1 ? 'project' : 'projects'}`}
            on:click={(e) => toggleBar(d, e)}
            on:keyup={(e) => toggleBar(d, e)}
          />
        {/each}

        <!-- Decorative annotation around the tallest bar.
             pointer-events: none so it never steals hover/click from the
             bar underneath. Dimmed together with the other bars whenever
             a non-max bar is the selected one. -->
        {#if maxBar}
          <g
            class="annotation-group"
            aria-hidden="true"
            style:opacity={selectedLabel === null || selectedLabel === maxBar.label ? 1 : 0.35}
          >
            <rect
              class="annotation-outline"
              x={xScale(maxBar.label) - 1.5}
              y={yScale(maxBar.value) - 1.5}
              width={xScale.bandwidth() + 3}
              height={innerHeight - yScale(maxBar.value) + 1.5}
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              rx="3"
              ry="3"
            />
            <line
              x1={xScale(maxBar.label) + xScale.bandwidth() + 4}
              y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
              x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
              y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
              stroke="currentColor"
              stroke-width="1.2"
            />
            <text
              x={xScale(maxBar.label) + xScale.bandwidth() + 34}
              y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
              dominant-baseline="middle"
              class="annotation"
            >
              Year with most projects
            </text>
          </g>
        {/if}

        <text
          x={innerWidth / 2}
          y={innerHeight + 50}
          text-anchor="middle"
          class="axis-label"
          aria-hidden="true"
        >
          Year
        </text>

        <text
          x={-(innerHeight / 2)}
          y={-margin.left + 18}
          text-anchor="middle"
          transform="rotate(-90)"
          class="axis-label"
          aria-hidden="true"
        >
          Number of Projects
        </text>
      </g>
    </svg>

    <ul class="legend" aria-hidden="true">
      {#each data as d}
        <li style="--color: {colorScale(d.label)}">
          <span class="swatch"></span>
          <span class="legend-label">{d.label}</span>
          <span class="legend-value">({d.value})</span>
        </li>
      {/each}
    </ul>
  </div>
{:else}
  <table class="data-table" aria-label="Table showing project counts by year">
    <caption>Projects by Year</caption>
    <thead>
      <tr>
        <th id="year-header" scope="col">Year</th>
        <th id="projects-header" scope="col">Projects</th>
      </tr>
    </thead>
    <tbody>
      {#each data as d, i}
        <tr>
          <th id="row-{i}" scope="row">{d.label}</th>
          <td aria-labelledby="row-{i} projects-header">{d.value}</td>
        </tr>
      {/each}
    </tbody>
  </table>
{/if}

<!-- ARIA live region: announces selection + view changes to screen readers -->
<p class="sr-only" aria-live="polite" aria-atomic="true">{liveText}</p>

<style>
  /* ---------- Toggle button ---------- */
  .toggle-button {
    width: auto;
    padding: 0.35rem 0.85rem;
    margin: 0 0 0.75rem;
    font-size: 0.88rem;
    border: 1px solid var(--border-gray);
    border-radius: 0.4rem;
    background: color-mix(in oklch, canvas 88%, canvastext 12%);
    color: canvastext;
    cursor: pointer;
  }

  .toggle-button:hover {
    background: color-mix(in oklch, var(--color-accent), canvas 85%);
  }

  .toggle-button:focus-visible {
    outline: 2px solid var(--color-accent);
    outline-offset: 2px;
  }

  /* ---------- Chart layout ---------- */
  .container {
    display: flex;
    align-items: start;
    gap: 0.75rem;
    margin: 0 0 1.75rem;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
    flex: 0 1 650px;
  }

  /* ---------- Bars ----------
     No opacity rule here — the opacity attribute on each rect is the
     single source of truth for dimming. CSS is only for stroke, cursor,
     and the focus-visible ring. */
  .bar {
    stroke-width: 1;
    cursor: pointer;
    outline: none;
    transition: opacity 250ms, stroke 150ms, stroke-width 150ms;
  }

  .bar:focus-visible {
    stroke: var(--color-accent);
    stroke-width: 2.5;
    stroke-dasharray: 4;
  }

  /* ---------- Annotation group ---------- */
  .annotation-group {
    pointer-events: none;
    transition: opacity 250ms;
  }

  /* ---------- SVG text (adaptive light/dark) ---------- */
  .chart-title {
    font-size: 1.15em;
    font-weight: 700;
    fill: currentColor;
    letter-spacing: -0.01em;
  }

  .axis-label {
    font-size: 0.78em;
    fill: currentColor;
    opacity: 0.72;
    font-weight: 500;
  }

  .annotation {
    font-size: 0.68em;
    fill: currentColor;
    opacity: 0.72;
    font-style: italic;
    font-weight: 500;
  }

  :global(.axis text) {
    font-size: 11px;
    fill: currentColor;
    opacity: 0.7;
  }

  /* ---------- Legend ---------- */
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
    border: 1px solid currentColor;
    flex-shrink: 0;
  }

  .legend-label {
    font-weight: 500;
  }

  .legend-value {
    opacity: 0.65;
    font-size: 0.9em;
  }

  /* ---------- Data table ---------- */
  .data-table {
    width: 100%;
    max-width: 30rem;
    margin: 0 0 1.75rem;
    border-collapse: collapse;
    font-size: 0.95rem;
  }

  .data-table caption {
    font-weight: 700;
    text-align: left;
    margin-bottom: 0.5rem;
    font-size: 1.05rem;
  }

  .data-table th,
  .data-table td {
    border: 1px solid var(--border-gray);
    padding: 0.5rem 0.75rem;
    text-align: left;
  }

  .data-table thead th {
    background: var(--surface);
    font-weight: 600;
  }

  .data-table tbody th {
    font-weight: 500;
    background: color-mix(in oklch, canvas 96%, canvastext 4%);
  }

  /* ---------- Screen-reader-only (for the live region) ---------- */
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }
</style>
