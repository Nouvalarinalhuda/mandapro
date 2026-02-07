<script>
  let { children } = $props();
  import "../app.css";
</script>
{@render children()}

<svelte:head>
  <title>MAN 2 Kota Probolinggo</title>
  <meta name="description" content="Website Resmi MAN 2 Kota Probolinggo - Madrasah Islami, Unggul, dan Berbasis Teknologi" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link rel="icon" href="/foto/kemenag.png" />

  <!-- SEO OG Tags -->
  <meta property="og:title" content="MAN 2 Kota Probolinggo" />
  <meta property="og:description" content="Website Resmi MAN 2 Kota Probolinggo - Islami, Unggul, Teknologi." />
  <meta property="og:image" content="/images/og-cover.jpg" />
  <meta property="og:url" content="https://man2kotaprobolinggo.sch.id/" />
  <meta name="twitter:card" content="summary_large_image" />
</svelte:head>
