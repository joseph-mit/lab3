<script>
  import { base } from "$app/paths";

  export let data = {};
</script>

<div class="reading-item">
  <img
    src={base + data.image}
    alt={data.title}
  />

  <div class="reading-text">
    <h3>{data.title}</h3>
    <p class="author">{data.author}</p>
  </div>
</div>

<style>
  .reading-item {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
  }

  img {
    width: 90px;
    height: auto;
    display: block;
    flex-shrink: 0;
  }

  .reading-text {
    display: grid;
    gap: 0.35rem;
  }

  h3 {
    margin: 0;
    font-size: 1.1rem;
  }

  .author {
    margin: 0;
    font-style: italic;
  }
</style>
