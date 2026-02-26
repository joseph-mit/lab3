<svelte:head>
  <title>Contact me</title>
</svelte:head>

<nav>
  <a href=".">Home</a>
  <a href="projects">Projects</a>
  <a href="contact" class="current">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/joseph-mit" target="_blank">GitHub</a>
</nav>

<h1>Contact</h1>

<form action="mailto:josephfn@mit.edu" method="GET">
  <label>
    Email:
    <input name="email" type="email" />
  </label>

  <br />

  <label>
    Subject:
    <input name="subject" />
  </label>

  <br />

  <label>
    Message:
    <textarea name="body"></textarea>
  </label>

  <br />

  <button>Send</button>
</form>
