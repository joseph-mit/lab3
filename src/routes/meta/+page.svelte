<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';
  import LineChart from '$lib/LineChart.svelte';
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
  let svg;
  let linesByDate = [];

  let hoveredCommit = {};
  $: if (hoveredIndex >= 0 && hoveredIndex < commits.length) {
    hoveredCommit = commits[hoveredIndex];
  }

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

  // Brushing
  let brushSelection = null;

  function brushed(evt) {
    brushSelection = evt.selection;
  }

  function isCommitBrushed(commit) {
    if (!brushSelection) return false;
    let [[x0, y0], [x1, y1]] = brushSelection;
    let cx = xScale(commit.datetime);
    let cy = yScale(commit.hourFrac);
    return cx >= x0 && cx <= x1 && cy >= y0 && cy <= y1;
  }

  $: {
    if (svg) {
      d3.select(svg).call(d3.brush()
        .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
        .on("start brush end", brushed));
      d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }
  }

  $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];
  $: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));

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

  // Bar chart data (filtered by both brushed and clicked)
  $: selectedLines =
    selectedCommits.length > 0
      ? selectedCommits.flatMap(c => c.lines)
      : locData;

  $: languageCounts = d3.rollup(selectedLines, v => v.length, d => d.type);
  $: allLanguages = Array.from(new Set(locData.map(d => d.type)));
  $: barData = allLanguages.map(lang => ({
    label: lang,
    value: languageCounts.get(lang) || 0
  }));

  $: barTitle =
    selectedCommits.length > 0
      ? `Lines of Code: ${selectedCommits.length} Selected Commits`
      : 'Lines of Code: Website Breakdown';

  // Line chart data wrangling
  $: {
    if (locData.length > 0) {
      let rolled = d3.rollups(locData, v => v.length, d => d3.timeDay.floor(d.datetime))
        .map(([date, count]) => ({ date, count }));
      let [minDate, maxDate] = d3.extent(rolled, d => d.date);
      let allDays = d3.timeDays(minDate, d3.timeDay.offset(maxDate, 1));
      linesByDate = allDays.map(date => ({
        date,
        count: rolled.find(r => r.date.getTime() === date.getTime())?.count ?? 0
      }));
    }
  }

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
        author, date, time, timezone, datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines
      };
    });

    commits = d3.sort(commits, d => -d.totalLines);
  });
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>
<p>Stats about the code of this website.</p>

{#if commits.length > 0}
  <dl class="stats">
    <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
    <dd>{locData.length}</dd>

    <dt>Commits</dt>
    <dd>{commits.length}</dd>

    <dt>Files</dt>
    <dd>{d3.groups(locData, d => d.file).length}</dd>

    <dt>Authors</dt>
    <dd>{d3.groups(locData, d => d.author).length}</dd>

    <dt>Max depth</dt>
    <dd>{d3.max(locData, d => d.depth)}</dd>

    <dt>Longest line</dt>
    <dd>{d3.max(locData, d => d.length)} chars</dd>
  </dl>
{/if}

<h2>Commits by time of day</h2>

<svg viewBox="0 0 {width} {height}" bind:this={svg}>
  <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
  <g transform="translate({usableArea.left}, 0)" bind:this={yAxisEl} />
  <g transform="translate(0, {usableArea.bottom})" bind:this={xAxisEl} />
  <g class="dots">
    {#each commits as commit, index (commit.id)}
      <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions a11y-mouse-events-have-key-events -->
      <circle
        cx={xScale(commit.datetime)}
        cy={yScale(commit.hourFrac)}
        r={rScale(commit.totalLines)}
        class="dot"
        class:selected={selectedCommits.includes(commit)}
        on:mouseenter={evt => dotInteraction(index, evt)}
        on:mouseleave={evt => dotInteraction(index, evt)}
        on:click={evt => dotInteraction(index, evt)}
      />
    {/each}
  </g>
</svg>

<dl
  class="info tooltip"
  hidden={hoveredIndex === -1}
  style="left: {tooltipPosition.x}px; top: {tooltipPosition.y}px"
  bind:this={commitTooltip}
>
  <dt>Commit</dt>
  <dd><a href={hoveredCommit.url} target="_blank">{hoveredCommit.id}</a></dd>
  <dt>Author</dt>
  <dd>{hoveredCommit.author}</dd>
  <dt>Date</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString('en', { dateStyle: 'full' })}</dd>
  <dt>Time</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString('en', { timeStyle: 'short' })}</dd>
  <dt>Lines</dt>
  <dd>{hoveredCommit.totalLines}</dd>
</dl>

{#if barData.length > 0}
  <BarHorizontal data={barData} title={barTitle} />
{/if}

{#if linesByDate.length > 0}
  <LineChart data={linesByDate} />
{/if}

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .gridlines {
    stroke-opacity: 0.2;
  }

  .dot {
    fill: steelblue;
    fill-opacity: 0.6;
    cursor: pointer;
    transition: 200ms;
    transform-origin: center;
    transform-box: fill-box;
  }

  .dot:hover {
    transform: scale(1.25);
    fill: darkgreen;
  }

  .dot.selected {
    fill: var(--color-accent);
    fill-opacity: 0.9;
  }

  .dot.selected:hover {
    fill: var(--color-accent);
  }

  @keyframes marching-ants {
    to { stroke-dashoffset: -8; }
  }

  svg :global(.selection) {
    fill-opacity: 10%;
    stroke: black;
    stroke-opacity: 70%;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
  }

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
    color: #888;
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
    background: white;
    box-shadow: 0 4px 18px rgba(0,0,0,0.15);
    border-radius: 0.6em;
    padding: 0.7em 1em;
    pointer-events: auto;
    transition-duration: 500ms;
    transition-property: opacity, visibility;
    z-index: 10;
  }

  .tooltip[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
  }

  :global(.axis text) {
    font-size: 11px;
    fill: #555;
  }

  .stats {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(8rem, 1fr));
    gap: 0.5rem 1.5rem;
    margin: 1rem 0 1.5rem;
    padding: 1rem 1.25rem;
    border-radius: 0.6em;
    background: var(--surface, #f5f5f5);
    border: 1px solid var(--border-gray, #ddd);
  }

  .stats dt {
    grid-row-end: span 2;
    font-size: 0.72em;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: #888;
    font-weight: 600;
    margin: 0;
    align-self: end;
  }

  .stats dd {
    margin: 0;
    font-size: 1.5em;
    font-weight: 700;
    line-height: 1.1;
  }
</style>
