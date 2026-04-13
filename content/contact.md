+++
date = '2026-04-13T13:37:00+02:00'
draft = false
title = 'Contact'
+++

<style>
.contact-form {
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
  max-width: 520px;
  margin-top: 1rem;
}

.contact-form .field {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.contact-form label {
  font-size: 0.9rem;
  opacity: 0.7;
}

.contact-form input,
.contact-form textarea {
  padding: 0.6rem 0.8rem;
  border-radius: 6px;
  border: 1px solid rgba(255,255,255,0.15);
  background: rgba(255,255,255,0.05);
  color: inherit;
  font-size: 1rem;
  font-family: inherit;
  outline: none;
  transition: border-color 0.2s;
}

.contact-form input:focus,
.contact-form textarea:focus {
  border-color: rgba(255,255,255,0.4);
}

.contact-form textarea {
  resize: vertical;
  min-height: 140px;
}

.contact-form button {
  align-self: flex-start;
  padding: 0.6rem 1.6rem;
  border-radius: 6px;
  border: 1px solid rgba(255,255,255,0.3);
  background: transparent;
  color: inherit;
  font-size: 1rem;
  font-family: inherit;
  cursor: pointer;
  transition: background 0.2s, border-color 0.2s;
}

.contact-form button:hover {
  background: rgba(255,255,255,0.1);
  border-color: rgba(255,255,255,0.6);
}
</style>

<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>

<form class="contact-form" action="https://forms.nvds.be/YOUR_FORM_KEY" method="POST">
  <input type="hidden" name="_redirect" value="https://nvds.be/contact/thanks">
  <input type="text" name="_hp_email" style="display:none">

  <div class="field">
    <label for="name">Name</label>
    <input type="text" id="name" name="name" required>
  </div>

  <div class="field">
    <label for="email">Email</label>
    <input type="email" id="email" name="email" required>
  </div>

  <div class="field">
    <label for="message">Message</label>
    <textarea id="message" name="message" required></textarea>
  </div>

  <div class="cf-turnstile" data-sitekey="0x4AAAAAAC8xs2eekzuHnbKe"></div>
  <button type="submit">Send</button>
</form>
