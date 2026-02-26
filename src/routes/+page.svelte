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

<div class="home-layout">
  <div class="about">
    <h1>Angel</h1>

    <p>
      Hi! I'm Angel, and I'm a green cheek conure. Some people say I'm a picky eater,
      but I prefer the term "professional food critic." When I'm not taste testing my
      humans' food, you can find me exploring kitchen cabinets or practicing my circus arts :)
    </p>

    <img
      src={base + "/images/pinot.png"}
      alt="Angel the conure"
    />
  </div>

  <section class="reading-section">
    <h2>What I'm Reading</h2>

    <div class="reading-list">
      {#each reading as item}
        <ReadingItem data={item} />
      {/each}
    </div>
  </section>
</div>

<h2>Latest projects</h2>

<div class="projects">
  {#each projects.slice(0, 3) as p}
    <Project data={p} />
  {/each}
</div>

<style>
  .home-layout {
    display: grid;
    grid-template-columns: 1.3fr 1fr;
    gap: 2rem;
    align-items: start;
  }

  .about img {
    display: block;
    width: 100%;
    max-width: 460px;
    height: auto;
  }

  .reading-section {
    background: #eadce1;
    padding: 1.5rem;
    border-radius: 0.75rem;
  }

  .reading-list {
    display: grid;
    gap: 1.25rem;
  }

  @media (max-width: 900px) {
    .home-layout {
      grid-template-columns: 1fr;
    }
  }
</style>
