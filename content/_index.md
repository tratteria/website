---
title: "Home"
---

<style>
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  font-family: 'Roboto', sans-serif;
  color: #fff;
  background-size: cover;
  background: #00796B;
}

.home {
  text-align: center;
  padding: 5em 0;
}

.home img {
  width: 300px;
  height: auto;
}

.home h1 {
  font-size: 3.5vw;
  font-weight: 700;
}

.home h2 {
  font-size: 2.5vw;
  font-weight: normal;
  margin-bottom: 30px;
}

.home h3 {
  font-size: 1.5vw;
  font-weight: normal;
  margin-bottom: 60px;
}

.home h3 a {
  color: #c94a4a;
  text-decoration: none;
  font-weight: 500;
}

.home .buttons {
  margin-top: 20px;
}

.buttons a {
  display: inline-block;
  margin: 0 1em;
  padding: 1em 3em;
  border-radius: 30px;
  background: #5a67d8;
  color: #fff;
  text-decoration: none;
  font-size: 1.2vw;
  font-weight: 500;
  transition: background 0.3s ease, transform 0.3s ease;
}

.buttons a:hover {
  background: #434190;
  transform: scale(1.05);
}


@media (max-width: 768px) {
  .home h1, .home h2, .home h3, .buttons a {
    font-size: 16px;
  }
  
  .home img {
    width: 50%;
  }

  .buttons a {
    padding: 10px 20px;
    font-size: 14px;
  }
}

</style>

<div class="home">
  <img src="/img/logos/image-logo.svg" alt="Tratteria Logo">
  <h1>Tratteria</h1>
  <h2>A Transaction Tokens Service</h2>
  <h3 style="margin-bottom: 0px;">Secure your microservices with <a href="https://datatracker.ietf.org/doc/draft-ietf-oauth-transaction-tokens/" target="_blank">Transaction Token</a>.</h3>
  <h3>Integrate into your microservices application with ease.</h3>
  <div class="buttons">
    <a href="/docs/quickstart" class="button"><i class="fas fa-rocket"></i> Get Started</a>
    <a href="/docs" class="button"><i class="fas fa-info-circle"></i> Learn More</a>
  </div>
</div>
