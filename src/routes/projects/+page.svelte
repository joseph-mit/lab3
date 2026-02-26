<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<nav>
  <a href=".">Home</a>
  <a href="projects" class="current">Projects</a>
  <a href="contact">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/joseph-mit" target="_blank">GitHub</a>
</nav>

<h1>Projects ({projects.length})</h1>

<div class="projects-grid">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>

<style>
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
