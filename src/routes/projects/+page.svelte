<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  const years = projects.map((p) => p.year);
  const minYear = Math.min(...years);
  const maxYear = Math.max(...years);
  const range = maxYear - minYear;
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>

<p class="intro">
  Scroll down to see a timeline of projects and what each one taught me along the way.
</p>

<ProjectNarrative />

<p class="outro">
  Thanks for reading the story. If you just want to browse, everything’s listed below.
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
