<script>
  import * as d3 from 'd3';
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";
  import Bar from '$lib/Bar.svelte';

  let years = projects.map(p => p.year);
  let range = Math.max(...years) - Math.min(...years);

  $: barData = d3.rollups(projects, v => v.length, d => d.year)
      .map(([year, count]) => ({ label: String(year), value: count }));
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<Bar data={barData} />

<h1>{projects.length} Projects over {range} Years</h1>

<p class="intro">
  Scroll down to see a timeline of my projects and how they've shaped what I know.
</p>

<ProjectNarrative />

<p class="outro">
  Thanks for scrolling! Here's the full list if you want to browse.
</p>

<div class="projects-grid">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>

<style>
  .intro {
    margin: 0 0 1.5rem;
  }

  .outro {
    margin: 2rem 0 1.75rem;
  }

  .projects-grid {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 2rem;
    align-items: start;
  }

  @media (max-width: 1200px) {
    .projects-grid {
      grid-template-columns: repeat(3, minmax(0, 1fr));
    }
  }

  @media (max-width: 900px) {
    .projects-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
    }
  }

  @media (max-width: 650px) {
    .projects-grid {
      grid-template-columns: 1fr;
    }
  }
</style>
