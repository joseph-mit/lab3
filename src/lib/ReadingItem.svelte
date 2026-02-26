<script>
  import { base } from "$app/paths";

  export let data = {};
</script>

<div class="reading-item">
  <img
    src={base + data.image}
    alt={data.title}
  />

  <div class="reading-copy">
    <h3>{data.title}</h3>
    <p class="author">{data.author}</p>
  </div>
</div>

<style>
  .reading-item {
    display: grid;
    grid-template-columns: 90px 1fr;
    gap: 1rem;
    align-items: start;
  }

  img {
    display: block;
    width: 90px;
    aspect-ratio: 2 / 3;
    object-fit: cover;
    border-radius: 0.35rem;
  }

  .reading-copy {
    display: grid;
    gap: 0.25rem;
  }

  h3 {
    margin: 0;
    font-size: 1.1rem;
    line-height: 1.2;
  }

  .author {
    margin: 0;
    font-style: italic;
  }
</style>
