---
layout: page
icon: fas fa-info-circle
order: 4
---

<section class="about-section">
  <div class="container">
    <div>
     <p><span class="command">$ whoami</span></p>
     <p id="scramble-text"> 👋 Hello! <span id="scramble-target">I'm Sameeh</span>,</p>
      <p>
        <span id="animated-text"></span>
      </p>
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
      Find my web application testing notes <a href="path/to/notes.pdf" target="_blank">here</a>.
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

  #animated-text::after {
    content: '|';
    display: inline-block;
    opacity: 1;
    animation: blink 0.7s infinite;
  }

@keyframes blink {
    0%, 100% { opacity: 1; }
    50% { opacity: 0; }
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

<script>
  function scrambleText(element, text, duration) {
    let scrambled = "";
    const characters = "!@#$%^&*()_+{}:<>?1234567890abcdefghijklmnopqrstuvwxyz";
    const textLength = text.length;
    const intervalTime = duration / textLength;
    let index = 0;

    const scrambleInterval = setInterval(() => {
      scrambled = text
        .split("")
        .map((char, i) => {
          if (i <= index) return char;
          return characters[Math.floor(Math.random() * characters.length)];
        })
        .join("");

      element.innerText = scrambled;

      if (index >= textLength) {
        clearInterval(scrambleInterval);
      }

      index++;
    }, intervalTime);
  }



  const roles = ["Pentester", "CTF player"];
  let roleIndex = 0;
  let charIndex = 0;
  const typingSpeed = 150;
  const erasingSpeed = 100;
  const delayBetween = 1000;

  function typeText() {
    const target = document.getElementById("animated-text");
    if (charIndex < roles[roleIndex].length) {
      target.textContent += roles[roleIndex].charAt(charIndex);
      charIndex++;
      setTimeout(typeText, typingSpeed);
    } else {
      setTimeout(eraseText, delayBetween);
    }
  }

  function eraseText() {
    const target = document.getElementById("animated-text");
    if (charIndex > 0) {
      target.textContent = roles[roleIndex].substring(0, charIndex - 1);
      charIndex--;
      setTimeout(eraseText, erasingSpeed);
    } else {
      roleIndex = (roleIndex + 1) % roles.length;
      setTimeout(typeText, typingSpeed);
    }
  }

  // Start the typing effect when the page loads
  document.addEventListener("DOMContentLoaded", function () {
    setTimeout(typeText, delayBetween);
  });

    // When the page loads, apply the scramble effect
  document.addEventListener("DOMContentLoaded", function () {
    const textElement = document.getElementById("scramble-target");
    const originalText = "I'm Sameeh";
    scrambleText(textElement, originalText, 2000); // 2.5 seconds duration
  });


</script>
