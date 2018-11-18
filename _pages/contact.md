---
title: "Contact"
permalink: /contact
excerpt: "Contact page of CodeNotFound."
---

You can get in touch with us using the form below.

<form class="page__form" action="https://formspree.io/codenotfound.com@gmail.com" method="POST">
  <ul>
    <li>
      <label for="name" style="margin:10px 0;">Your name</label>
      <input type="text" id="name" name="name" placeholder="Your name">
    </li>
    <li>
      <label for="email" style="margin:10px 0;">Your email*</label>
      <input type="email" id="email" name="_replyto" placeholder="Your email" required>
    </li>
    <li class="hidden">
      <input type="hidden" name="_subject" value="New submission!" />
    </li>
    <li>
      <label for="message" style="margin:10px 0;">Your message*</label>
      <textarea id="message" name="message" spellcheck="true" rows="5" cols="50" placeholder="Your message" required></textarea>
    </li>
    <li>
      <input type="submit" value="Send message" class="btn btn--large btn--warning" style="margin:10px 0;">
    </li>
  </ul>
</form>
