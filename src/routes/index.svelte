<style>
  h1,
  figure,
  p {
    text-align: center;
    margin: 0 auto;
  }

  h1 {
    font-size: 2.8em;
    text-transform: uppercase;
    font-weight: 700;
    margin: 0 0 0.5em 0;
  }

  figure {
    margin: 0 0 1em 0;
  }

  img {
    width: 100%;
    max-width: 400px;
    margin: 0 0 1em 0;
  }

  p {
    margin: 1em auto;
  }

  @media (min-width: 480px) {
    h1 {
      font-size: 4em;
    }
  }
</style>

<svelte:head>
  <title>Sapper x Firebase</title>
  <meta
    name="description"
    content="Sapper hosted on Firebase via Functions - example (link to article
    included)." />
  <meta name="author" content="Eckhardt Dreyer" />
  <meta
    name="tags"
    content="Firebase and Sapper, Host sapper on Firebase, Sapper firebase
    functions, Firebase, Sapper, SSR, Sapper SSR Firebase" />
  <!-- Twitter Card data -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:site" content="@eckhardtdreyer" />
  <meta name="twitter:title" content="Sapper x Firebase" />
  <meta
    name="twitter:description"
    content="Sapper hosted on Firebase via Functions - example (link to article
    included)." />
  <meta name="twitter:creator" content="@eckhardtdreyer" />
  <meta
    name="twitter:image"
    content="https://sapper-firebase.firebaseapp.com/sapperfirebase.jpg" />

  <!-- Open Graph data -->
  <meta property="og:title" content="Sapper x Firebase" />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="http://www.example.com/" />
  <meta
    property="og:image"
    content="https://sapper-firebase.firebaseapp.com/sapperfirebase.jpg" />
  <meta
    property="og:image:secure_url"
    content="https://sapper-firebase.firebaseapp.com/sapperfirebase.jpg" />
  <meta
    property="og:description"
    content="Sapper hosted on Firebase via Functions - example (link to article
    included)." />
  <meta property="og:site_name" content="Sapper x Firebase" />
</svelte:head>

<h1>Great success!</h1>

<figure>
  <img alt="Borat" src="great-success.png" />
  <figcaption>HIGH FIVE!</figcaption>
</figure>

<p>
  <strong>Check out this page source, to see SSR content from Firebase</strong>
</p>
<br />
<br />
<p>
  <strong>
    Created by
    <a href="https://twitter.com/eckhardtdreyer">@eckhardtdreyer</a>
  </strong>
</p>

<h2>
  full article on
  <a
    href="https://dev.to/eckhardtd/how-to-host-a-sapper-js-ssr-app-on-firebase-hmb">
    Dev.to
  </a>
</h2>
