<script>
  import { base } from "$app/paths";
  import projects from "$lib/projects.json";
  import reading from "$lib/reading.json";
  import Project from "$lib/Project.svelte";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<nav>
  <a href="." class="current">Home</a>
  <a href="projects">Projects</a>
  <a href="contact">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/joseph-mit" target="_blank">GitHub</a>
</nav>

<h1>Joseph</h1>

<p>I am a graduate student at MIT.</p>

<img
  src={base + "/images/pinot.png"}
  alt="Enjoy a picture of Pinot smelling tulips last spring"
/>

<section>
  <h2>What I'm Reading</h2>

  <div class="reading">
    {#each reading as item}
      <ReadingItem data={item} />
    {/each}
  </div>
</section>

<h2>Latest projects</h2>

<div class="projects">
  {#each projects.slice(0, 3) as p}
    <Project data={p} />
  {/each}
</div>
