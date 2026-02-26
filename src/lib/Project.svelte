<script>
  import { base } from "$app/paths";

  export let data = {};
</script>

<article class="project-card">
  <h2>{data.title}</h2>

  <img
    src={base + data.image}
    alt={data.title}
  />

  <p>{data.description}</p>
</article>

<style>
  .project-card {
    display: grid;
    gap: 0.9rem;
    align-content: start;
  }

  h2 {
    margin: 0;
    font-size: 1.5rem;
    line-height: 1.15;
  }

  img {
    display: block;
    width: 100%;
    aspect-ratio: 4 / 3;
    object-fit: cover;
    border-radius: 0.5rem;
  }

  p {
    margin: 0;
    line-height: 1.5;
  }
</style>
