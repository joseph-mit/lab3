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

<div class="home-top">
  <section class="intro">
    <h1>Joseph</h1>

    <p>I am a graduate student at MIT.</p>

    <img
      src={base + "/images/pinot.png"}
      alt="Enjoy a picture of Pinot smelling tulips last spring"
    />
  </section>

  <section class="reading-panel">
    <h2>What I'm Reading</h2>

    <div class="reading-list">
      {#each reading as item}
        <ReadingItem data={item} />
      {/each}
    </div>
  </section>
</div>

<section class="latest-projects">
  <h2>Latest projects</h2>

  <div class="projects-grid">
    {#each projects.slice(0, 3) as p}
      <Project data={p} />
    {/each}
  </div>
</section>

<style>
  .home-top {
    display: grid;
    grid-template-columns: minmax(0, 1.35fr) minmax(320px, 0.95fr);
    gap: 2rem;
    align-items: start;
    margin-bottom: 2.75rem;
  }

  .intro img {
    display: block;
    width: 100%;
    max-width: 560px;
    height: auto;
    border-radius: 0.5rem;
  }

  .reading-panel {
    background: #eadfe4;
    padding: 1.5rem;
    border-radius: 0.9rem;
  }

  .reading-panel h2 {
    margin-top: 0;
  }

  .reading-list {
    display: grid;
    gap: 1.25rem;
  }

  .latest-projects h2 {
    margin-bottom: 1rem;
  }

  .projects-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    align-items: start;
  }

  @media (max-width: 900px) {
    .home-top {
      grid-template-columns: 1fr;
    }
  }
</style>
