<script>
  import projects from "$lib/projects.json";
  import { base } from "$app/paths";
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

<h1>Projects</h1>

<section id="projects">
  {#each projects as p}
    <article>
      <h2>Campus Mobility Map</h2>
      <img
        src={base + "/images/pinot.png"}
        alt="Project thumbnail placeholder"
      />
      <p>A quick explorable map prototype for commuting patterns around Cambridge.</p>
    </article>
  {/each}
</section>
