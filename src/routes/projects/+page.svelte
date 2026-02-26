

<svelte:head>
  <title>Projects</title>
</svelte:head>

<nav>
  <a href=".">Home</a>
  <a href="projects" class="current">Projects</a>
  <a href="contact">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/joseph-mit" target="_blank">GitHub</a>
</nav>

<h1>Projects</h1>

<section id="projects">
  <article>
    <h2>Campus Mobility Map</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A quick explorable map prototype for commuting patterns around Cambridge.</p>
  </article>

  <article>
    <h2>Housing Affordability Explorer</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>Small multiples for rent burden and income bands, with filters for neighborhood.</p>
  </article>
  <article>
    <h2>Real Estate Carbon Budget Dashboard</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A simple view comparing operational vs embodied emissions under retrofit scenarios.</p>
  </article>

  <article>
    <h2>Transit Catchment Story Map</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A narrative map showing walk sheds and transfer friction across key stations.</p>
  </article>

  <article>
    <h2>Solar Potential Quick Look</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A lightweight calculator to sanity-check rooftop PV capacity by building type.</p>
  </article>

  <article>
    <h2>Retail Footfall Heatmap</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>Weekday vs weekend foot traffic patterns for a corridor, grouped by hour.</p>
  </article>

  <article>
    <h2>Zoning Change Timeline</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A clean timeline of rezoning events, with quick links to the underlying documents.</p>
  </article>

  <article>
    <h2>Flood Risk Parcel Viewer</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>Parcel-level lookup for flood zones and elevation, built for fast “what if” checks.</p>
  </article>

  <article>
    <h2>Construction Cost Sensitivity Tool</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A small sensitivity grid for hard-cost inflation, interest rates, and stabilized NOI.</p>
  </article>

  <article>
    <h2>Data Center Load Monitor</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>Sketch of an interface to track interconnection requests and projected MW demand.</p>
  </article>

  <article>
    <h2>Noise Complaints Analyzer</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A quick pass at cleaning and summarizing complaint text by topic and geography.</p>
  </article>

  <article>
    <h2>Street Tree Equity Audit</h2>
    <img
      src={asset('/images/pinot.png')}
      alt="Project thumbnail placeholder"
    />
    <p>A simple comparison of canopy coverage vs heat exposure across census tracts.</p>
  </article>
</section>
