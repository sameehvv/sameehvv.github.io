---
layout: page
icon: fas fa-info-circle
order: 4
---

<section class="about-section">
  <div class="container">
    <div>
     <p><span class="command">$ whoami</span></p>
     <p id="scramble-text"> 👋 Hello! <span>I'm Sameeh</span>,</p>
      <p>Pentester | CTF player</p>
    </div>
    <div>
      <p><span class="command">$ ls /certs</span></p>
      <div class="certifications">
        <img src="/assets/img/certs/bscp.png" alt="BSCP" title="BSCP">
        <img src="/assets/img/certs/ecppt.png" alt="eCPPT" title="eCPPT">
        <img src="/assets/img/certs/ceh.png" alt="CEH" title="CEH">
      </div>
    </div>
    <div>
      <p><span class="command">$ ls /badges</span></p>
      <div class="badges">
        <img src="https://tryhackme-badges.s3.amazonaws.com/Sameeh.png" alt="TryHackMe Badge" class="badge-image">
        <img src="https://www.hackthebox.com/badge/image/440500" alt="HackTheBox Badge" class="badge-image">
      </div>
    </div>
    <div>
      <p><span class="command">$ cat notes.txt</span></p>
        <p>
      Find my web application testing notes <a href="https://zesty-industry-7a0.notion.site/Web-Pentest-1fbddf00cb2e4571a90433b3713984d3" target="_blank">here</a>.
        </p>
    </div>
  </div>
</section>

<style>
  .about-section {
    font-family: 'Courier New', monospace;
    line-height: 1.8;
  }

  .command {
    color: green;
    font-family: 'Courier New', monospace;
  }

  .certifications img {
    width: 70px;
    height: auto;
    margin-right: 15px;
  }


  .badges img {
    width: 200px;
    height: auto;
    margin-right: 15px;
  }

  .certifications, .badges {
    display: flex;
    justify-content: start;
    align-items: center;
    gap: 35px;
    margin-top: 15px;
    margin-bottom: 10px;
  }

  .notes-link {
    color: #1a73e8;
    text-decoration: none;
    font-weight: bold;
  }

  .notes-link:hover {
    text-decoration: underline;
  }

@media (max-width: 768px) {
    .about-section {
      padding: 20px;
    }

    .certifications img {
      width: 60px;
    }
    
    .badges img {
      width: 200px; 
    }

    .certifications, .badges {
      gap: 20px;
    }
  }
</style>
