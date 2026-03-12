<script>
  import { base } from "$app/paths";
  import Scrolly from "svelte-scrolly";
  import projects from "$lib/projects.json";

  let scrollyProgress = 0;

  // Sort without mutating the original array (it avoids weird surprises later).
  const sorted_projects = projects.slice().sort((a, b) => a.year - b.year);

  const progressPerProject = 100 / sorted_projects.length;

  $: activeProjectIdx = Math.min(
    sorted_projects.length - 1,
    Math.floor(scrollyProgress / progressPerProject)
  );

  $: activeProject = sorted_projects[activeProjectIdx];
</script>

<div class="scrolly-wrapper">
  <Scrolly bind:progress={scrollyProgress}>
    <section class="steps">
      {#each sorted_projects as p}
        <section class="step">
          <div class="step-content">
            <h3>{p.year} — {p.title}</h3>
            <p>{p.story}</p>
          </div>
        </section>
      {/each}
    </section>

    <div slot="viz" class="project-detail">
      {#if activeProject}
        <h3 class="project-year">{activeProject.year}</h3>
        <img
          class="project-image"
          src={base + activeProject.image}
          alt={`Image for ${activeProject.title}`}
        />
        <p class="project-title">{activeProject.title}</p>
        <p class="project-desc">{activeProject.description}</p>
      {/if}
    </div>
  </Scrolly>
</div>

<style>
  .scrolly-wrapper {
    width: min(1000px, 92vw);
    position: relative;
    left: 50%;
    transform: translateX(-50%);
    margin: 2rem 0 3rem;
  }

  .step {
    min-height: 80vh;
    padding: 2rem 0;
  }

  .step-content {
    border-left: 4px solid var(--color-accent);
    padding: 1.25rem 1.75rem;
    background: color-mix(in oklch, canvas 92%, canvastext 8%);
    border-radius: 0.75rem;
  }

  .step-content h3 {
    margin: 0 0 0.6rem;
    line-height: 1.15;
  }

  .step-content p {
    margin: 0;
  }

  .project-detail {
    padding: 2rem;
    width: 100%;
  }

  .project-year {
    margin: 0 0 0.75rem;
    font-size: 1.1rem;
    color: color-mix(in oklch, canvastext 70%, canvas);
  }

  .project-image {
    width: 100%;
    max-width: 520px;
    height: auto;
    border-radius: 0.9rem;
    border: 1px solid var(--border-gray);
    display: block;
  }

  .project-title {
    margin: 0.9rem 0 0.25rem;
    font-weight: 700;
  }

  .project-desc {
    margin: 0;
    color: color-mix(in oklch, canvastext 70%, canvas);
  }

  @media (max-width: 900px) {
    .scrolly-wrapper {
      width: min(900px, 94vw);
    }

    .project-detail {
      padding: 1.25rem 0;
    }
  }
</style>
