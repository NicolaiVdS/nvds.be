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

/* ── Light mode ── */
[data-theme="light"] .contact-form input,
[data-theme="light"] .contact-form textarea {
  border: 1px solid rgba(0, 0, 0, 0.15);
  background: rgba(0, 0, 0, 0.03);
}
[data-theme="light"] .contact-form input:focus,
[data-theme="light"] .contact-form textarea:focus {
  border-color: rgba(0, 0, 0, 0.35);
  background: rgba(0, 0, 0, 0.05);
}
[data-theme="light"] .contact-form button {
  border-color: rgba(0, 0, 0, 0.2);
}
[data-theme="light"] .contact-form button:hover {
  background: rgba(0, 0, 0, 0.05);
  border-color: rgba(0, 0, 0, 0.4);
}

/* Dark mode (default) */
cap-widget {
  --cap-background: #2a2a2a;
  --cap-border-color: rgba(255, 255, 255, 0.1);
  --cap-border-radius: 8px;
  --cap-widget-height: 80px;
  --cap-widget-width: 230px;
  --cap-widget-padding: 14px;
  --cap-gap: 15px;
  --cap-color: #e8e8e8;
  --cap-checkbox-size: 22px;
  --cap-checkbox-border: 1px solid rgba(255, 255, 255, 0.2);
  --cap-checkbox-border-radius: 5px;
  --cap-checkbox-background: rgba(255, 255, 255, 0.05);
  --cap-checkbox-margin: 2px;
  --cap-font: system-ui, -apple-system, sans-serif;
  --cap-spinner-color: #e8e8e8;
  --cap-spinner-background-color: rgba(255, 255, 255, 0.1);
  --cap-spinner-thickness: 3px;
}

/* Light mode */
[data-theme="light"] cap-widget {
  --cap-background: #f4f4f4;
  --cap-border-color: rgba(0, 0, 0, 0.1);
  --cap-color: #1a1a1a;
  --cap-checkbox-border: 1px solid rgba(0, 0, 0, 0.2);
  --cap-checkbox-border-radius: 5px;
  --cap-checkbox-background: rgba(0, 0, 0, 0.03);
  --cap-spinner-color: #1a1a1a;
  --cap-spinner-background-color: rgba(0, 0, 0, 0.08);
}
</style>

<script src="https://captcha.nvds.be/assets/widget.js"></script>
<script src="https://captcha.nvds.be/assets/floating.js"></script> 

<form id="contact-form" class="contact-form" action="https://api.nvds.be/v1/contact" method="POST">
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

  <cap-widget id="cap" data-cap-api-endpoint="https://captcha.nvds.be/fcf0d0856d/"></cap-widget>
  <button type="button" id="submit-btn" data-cap-floating="#cap"  data-cap-floating-position="bottom">Send</button>
</form>

<div id="thank-you" style="display:none; margin-top:1rem;">
  <h3>Thanks!</h3>
  <p>Your message has been sent successfully.</p>
</div>

<script>
const form = document.getElementById('contact-form');
const capWidget = document.getElementById('cap');
const submitBtn = document.getElementById('submit-btn');
const thankYou = document.getElementById('thank-you');

function setFormDisabled(state) {
    [...form.elements].forEach(el => el.disabled = state);
    submitBtn.disabled = state;
}

async function sendForm() {
    const formData = new FormData(form);

    setFormDisabled(true);
    submitBtn.textContent = "Sending...";

    try {
        const res = await fetch(form.action, {
            method: "POST",
            headers: {
                "Content-Type": "application/x-www-form-urlencoded"
            },
            body: new URLSearchParams(formData).toString()
        });

        if (!res.ok) {
            throw new Error("Request failed");
        }

        // success UI
        form.style.display = "none";
        thankYou.style.display = "block";

    } catch (err) {
        alert("Failed to send message. Please try again.");
        setFormDisabled(false);
        submitBtn.textContent = "Send";
    }
}

capWidget.addEventListener('solve', function (event) {
    if (form.checkValidity()) {
        sendForm();
    } else {
        form.reportValidity();
    }
});
</script>
