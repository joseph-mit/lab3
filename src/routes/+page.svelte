<script>
  import { base } from "$app/paths";
  import { onMount } from "svelte";

  import projects from "$lib/projects.json";
  import reading from "$lib/reading.json";
  import Project from "$lib/Project.svelte";
  import ReadingItem from "$lib/ReadingItem.svelte";

  const GITHUB_USERNAME = "joseph-mit";

  let githubData = null;
  let loading = true;
  let error = null;

  onMount(async () => {
    loading = true;
    error = null;

    const cacheKey = `github:${GITHUB_USERNAME}`;
    const cachedRaw = globalThis?.localStorage?.getItem(cacheKey);

    if (cachedRaw) {
      try {
        const cached = JSON.parse(cachedRaw);
        githubData = cached.data;
        loading = false;
      } catch {
        /* corrupted cache, ignore */
      }
    }

    try {
      const USE_FAKE_API = false;

      const response = USE_FAKE_API
        ? await {
            ok: true,
            json: async () => ({
              followers: 123,
              following: 45,
              public_repos: 67,
              public_gists: 8
            })
          }
        : await fetch(`https://api.github.com/users/${GITHUB_USERNAME}`);

      if (!response.ok) {
        throw new Error(`GitHub API error (${response.status})`);
      }

      const data = await response.json();
      githubData = data;

      globalThis?.localStorage?.setItem(
        cacheKey,
        JSON.stringify({ savedAt: Date.now(), data })
      );
    } catch (err) {
      error = err;
      if (!githubData) loading = false;
      return;
    }

    loading = false;
  });
</script>

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

{#if loading}
  <section class="github-stats">
    <h2>My GitHub Stats</h2>
    <p class="muted">Loading…</p>
  </section>
{:else if error}
  <section class="github-stats">
    <h2>My GitHub Stats</h2>
    <p class="error">Something went wrong: {error.message}</p>
  </section>
{:else if githubData}
  <section class="github-stats">
    <h2>My GitHub Stats</h2>

    <dl>
      <dt>Followers</dt>
      <dd>{githubData.followers}</dd>

      <dt>Following</dt>
      <dd>{githubData.following}</dd>

      <dt>Public Repos</dt>
      <dd>{githubData.public_repos}</dd>

      <dt>Public Gists</dt>
      <dd>{githubData.public_gists}</dd>
    </dl>
  </section>
{/if}

<section class="latest-projects">
  <h2>Latest Projects</h2>

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
    margin-bottom: 2.25rem;
  }

  .intro img {
    display: block;
    width: 100%;
    max-width: 560px;
    height: auto;
    border-radius: 0.5rem;
  }

  .reading-panel {
    background: color-mix(in oklch, var(--color-accent), canvas 88%);
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

  .github-stats {
    margin: 1.75rem 0 2.25rem;
    padding: 1.5rem;
    border: 1px solid var(--border-gray);
    border-radius: 0.9rem;
    background: color-mix(in oklch, canvas 92%, canvastext 8%);
  }

  .github-stats h2 {
    margin: 0 0 1rem;
  }

  .github-stats dl {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 0.5rem 1.25rem;
    margin: 0;
  }

  .github-stats dt {
    grid-row: 1;
    font-size: 0.85rem;
    letter-spacing: 0.04em;
    text-transform: uppercase;
    color: color-mix(in oklch, canvastext 65%, canvas);
  }

  .github-stats dd {
    grid-row: 2;
    margin: 0;
    font-size: 2rem;
    line-height: 1.05;
  }

  .muted {
    margin: 0;
    color: color-mix(in oklch, canvastext 65%, canvas);
  }

  .error {
    margin: 0;
  }

  .latest-projects {
    margin-top: 2.25rem;
  }

  .latest-projects h2 {
    margin-bottom: 1.25rem;
    font-size: 2rem;
    line-height: 1.1;
  }

  .projects-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 2rem;
    align-items: start;
  }

  @media (max-width: 900px) {
    .home-top {
      grid-template-columns: 1fr;
    }

    .projects-grid {
      grid-template-columns: 1fr;
    }

    .github-stats dl {
      grid-template-columns: repeat(2, minmax(0, 1fr));
      row-gap: 1rem;
    }

    .github-stats dt {
      grid-row: auto;
    }

    .github-stats dd {
      grid-row: auto;
    }
  }
</style>
