<script>
  import { base } from "$app/paths";
  export let data;
</script>

<div class="project">
  <h2>{data.title}</h2>
  <p class="year">{data.year}</p>

  <img src={base + data.image} alt={data.title} />

  <p class="description">{data.description}</p>
</div>

<style>
  .project img {
    width: 100%;
    height: auto;
    border-radius: 0.75rem;
  }

  .year {
    margin: 0.25rem 0 0.75rem;
    font-size: 0.95rem;
    color: color-mix(in oklch, canvastext 60%, canvas);
  }

  .description {
    margin: 0.75rem 0 0;
  }
</style>
