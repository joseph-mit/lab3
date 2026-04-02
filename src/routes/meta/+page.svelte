<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';
  import {
    computePosition,
    autoPlacement,
    offset
  } from '@floating-ui/dom';

  let locData = [];
  let commits = [];
  let clickedCommits = [];
  let hoveredIndex = -1;
  let tooltipPosition = { x: 0, y: 0 };
  let commitTooltip;

  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

  /* ── SVG dimensions ── */
  let width = 1000;
  let height = 600;
  let margin = { top: 30, right: 20, bottom: 40, left: 65 };

  $: usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
    width: width - margin.left - margin.right,
    height: height - margin.top - margin.bottom
  };

  /* ── Scales ── */
  $: xScale = d3.scaleTime()
    .domain(d3.extent(commits, d => d.datetime) ?? [new Date(), new Date()])
    .range([usableArea.left, usableArea.right])
    .nice();

  $: yScale = d3.scaleLinear()
    .domain([0, 24])
    .range([usableArea.top, usableArea.bottom]);

  $: rScale = d3.scaleSqrt()
    .domain(d3.extent(commits, d => d.totalLines) ?? [0, 1])
    .range([5, 30]);

  /* ── Axes (imperative D3) ── */
  let xAxisEl, yAxisEl, yAxisGridlines;

  $: if (xAxisEl) {
    d3.select(xAxisEl).call(d3.axisBottom(xScale));
  }

  $: if (yAxisEl) {
    d3.select(yAxisEl).call(
      d3.axisLeft(yScale)
        .tickFormat(d => String(d % 24).padStart(2, '0') + ':00')
    );
  }

  $: if (yAxisGridlines) {
    d3.select(yAxisGridlines).call(
      d3.axisLeft(yScale)
        .tickFormat('')
        .tickSize(-usableArea.width)
    );
  }

  /* ── Interaction helpers ── */
  function isSelected(commit) {
    return clickedCommits.includes(commit);
  }

  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;

    if (evt.type === 'mouseenter') {
      hoveredIndex = index;

      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: 'fixed',
        middleware: [offset(5), autoPlacement()]
      });
    } else if (evt.type === 'mouseleave') {
      hoveredIndex = -1;
    } else if (evt.type === 'click') {
      let commit = commits[index];
      if (!clickedCommits.includes(commit)) {
        clickedCommits = [...clickedCommits, commit];
      } else {
        clickedCommits = clickedCommits.filter(c => c !== commit);
      }
    }
  }

  /* ── Bar-chart data (filtered by selection) ── */
  $: selectedLines =
    clickedCommits.length > 0
      ? clickedCommits.flatMap(c => c.lines)
      : locData;

  $: languageCounts = d3.rollup(selectedLines, v => v.length, d => d.type);

  $: allLanguages = Array.from(new Set(locData.map(d => d.type)));

  $: barData = allLanguages.map(lang => ({
    label: lang,
    value: languageCounts.get(lang) || 0
  }));

  $: barTitle =
    clickedCommits.length > 0
      ? 'Lines of Code: Selected Commits'
      : 'Lines of Code: Website Breakdown';

  /* ── Load data ── */
  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
      ...row,
      line: Number(row.line),
      depth: Number(row.depth),
      length: Number(row.length),
      date: new Date(row.date + 'T00:00' + row.timezone),
      datetime: new Date(row.datetime)
    }));

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
      let first = lines[0];
      let { author, date, time, timezone, datetime } = first;
      return {
        id: commit,
        url: 'https://github.com/joseph-mit/lab-7/commit/' + commit,
        author,
        date,
        time,
        timezone,
        datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines
      };
    });

    /* Sort largest-first so small dots paint on top */
    commits = d3.sort(commits, d => -d.totalLines);
  });
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>
<p>Stats about the code of this website.</p>

<!-- ═══════ SCATTER PLOT ═══════ -->
<h2>Commits by time of day</h2>

<svg viewBox="0 0 {width} {height}">
  <!-- Grid lines (paint first = behind everything) -->
  <g
    class="gridlines"
    transform="translate({usableArea.left}, 0)"
    bind:this={yAxisGridlines}
  />

  <!-- Y axis -->
  <g
    transform="translate({usableArea.left}, 0)"
    bind:this={yAxisEl}
  />

  <!-- X axis -->
  <g
    transform="translate(0, {usableArea.bottom})"
    bind:this={xAxisEl}
  />

  <!-- Dots -->
  <g class="dots">
    {#each commits as commit, index}
      <circle
        cx={xScale(commit.datetime)}
        cy={yScale(commit.hourFrac)}
        r={rScale(commit.totalLines)}
        fill="steelblue"
        fill-opacity="0.6"
        class:selected={isSelected(commit)}
        on:mouseenter={evt => dotInteraction(index, evt)}
        on:mouseleave={evt => dotInteraction(index, evt)}
        on:click={evt => dotInteraction(index, evt)}
      />
    {/each}
  </g>
</svg>

<!-- ═══════ TOOLTIP ═══════ -->
<dl
  class="info tooltip"
  hidden={hoveredIndex === -1}
  style="left: {tooltipPosition.x}px; top: {tooltipPosition.y}px"
  bind:this={commitTooltip}
>
  <dt>Commit</dt>
  <dd>
    <a href={hoveredCommit.url} target="_blank">{hoveredCommit.id}</a>
  </dd>

  <dt>Author</dt>
  <dd>{hoveredCommit.author}</dd>

  <dt>Date</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString('en', { dateStyle: 'full' })}</dd>

  <dt>Time</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString('en', { timeStyle: 'short' })}</dd>

  <dt>Lines</dt>
  <dd>{hoveredCommit.totalLines}</dd>
</dl>

<!-- ═══════ BAR CHART ═══════ -->
{#if barData.length > 0}
  <BarHorizontal data={barData} title={barTitle} />
{/if}

<style>
  /* ── SVG container ── */
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  /* ── Grid lines ── */
  .gridlines {
    stroke-opacity: 0.2;
  }

  /* ── Circles ── */
  circle {
    cursor: pointer;
    transition: 200ms;
    transform-origin: center;
    transform-box: fill-box;
  }

  circle:hover {
    transform: scale(1.25);
    fill: oklch(60% 45% 250);
  }

  .selected {
    fill: var(--color-accent);
  }

  @keyframes marching-ants {
    to { stroke-dashoffset: -8; }
  }

  .selected {
    stroke: var(--color-accent);
    stroke-width: 2;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
    fill-opacity: 0.9;
  }

  /* ── Tooltip ── */
  dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 0.15em 0.75em;
    margin: 0;
    padding: 0;
  }

  dl.info dt {
    font-weight: 600;
    text-transform: uppercase;
    font-size: 0.75em;
    color: #666;
    margin: 0;
    text-align: right;
  }

  dl.info dd {
    margin: 0;
    font-size: 0.88em;
  }

  .tooltip {
    position: fixed;
    top: 1em;
    left: 1em;
    background: oklch(100% 0% 0 / 92%);
    box-shadow: 0 4px 18px oklch(0% 0 0 / 0.15);
    border-radius: 0.6em;
    padding: 0.7em 1em;
    backdrop-filter: blur(8px);
    pointer-events: auto;
    transition-duration: 500ms;
    transition-property: opacity, visibility;
    z-index: 10;
  }

  .tooltip[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
  }

  /* ── Axis styling ── */
  :global(.axis text) {
    font-size: 11px;
    fill: #555;
  }
</style>
