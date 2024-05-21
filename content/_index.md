---
title: "Home"
---

<style>
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  color: #fff;
  background: url('/img/background/image.png');
  background-attachment: fixed;
}

.home {
  text-align: center;
  padding: 4em 1em 6em 1em;
}

.home img {
  width: 350px;
  height: auto;
}

.home h1 {
  font-size: 4vw;
  font-weight: 800;
  margin-bottom: 20px;
}

.home h2 {
  font-size: 2.8vw;
  font-weight: 600;
  margin-bottom: 30px;
}

.home h3 {
  font-size: 1.8vw;
  font-weight: 400;
  margin-bottom: 50px;
}

.home h3 a {
  color: #ffd700;
  font-weight: 700;
  border-radius: 5px;
  transition: background-color 0.3s ease, color 0.3s ease;
  font-size: 1em;
}

.home h3 a:hover {
  color: #fff;
}

.home .buttons {
  margin-top: 30px;
}

.buttons a {
  display: inline-block;
  margin: 0 1em;
  padding: 1.2em 3em;
  border-radius: 30px;
  background: #5a67d8;
  color: #fff;
  text-decoration: none;
  font-size: 1.4vw;
  font-weight: 600;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: background 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
}

.buttons a:hover {
  background: #434190;
  transform: scale(1.05);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
}

footer {
  background: #333;
  padding: 2em 0;
  text-align: center;
  color: #fff;
}

footer a {
  color: #5a67d8;
  text-decoration: none;
}

@media (max-width: 768px) {
  .home h1, .home h2, .home h3, .buttons a {
    font-size: 5vw;
  }
  
  .home img {
    width: 70%;
  }

  .buttons a {
    padding: 10px 20px;
    font-size: 16px;
  }
}

</style>

<div class="home">
  <img src="/img/logos/image-logo.svg" alt="Tratteria Logo">
  <h1>Tratteria</h1>
  <h2>Transaction Tokens Service</h2>
  <h3>Secure your microservices with <a href="https://www.ietf.org/archive/id/draft-ietf-oauth-transaction-tokens-01.html" target="_blank">Transaction Token</a>.</h3>
  <div class="buttons">
    <a href="/docs/quickstart" class="button"><i class="fas fa-rocket"></i> Get Started</a>
    <a href="/docs" class="button"><i class="fas fa-info-circle"></i> Learn More</a>
  </div>
</div>