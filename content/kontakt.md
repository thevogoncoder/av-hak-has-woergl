---
title: "Kontakt"
showDate: false
showTableOfContents: false
showReadingTime: false
showWordCount: false
build:
  list: never
---

Haben Sie Anregungen oder möchten Sie Ihre E-Mail-Adresse bekanntgeben bzw. aktualisieren? Dann nehmen Sie bitte Kontakt mit uns auf.

---

## Kontaktformular

<form name="kontakt" method="POST" data-netlify="true" netlify-honeypot="bot-field" class="mt-4 space-y-4">
  <p class="hidden">
    <label>Nicht ausfüllen: <input name="bot-field" /></label>
  </p>
  <p>
    <label class="block font-bold mb-1" for="name">Name *</label>
    <input type="text" id="name" name="name" required
      class="w-full rounded border border-neutral-300 dark:border-neutral-600 bg-neutral-50 dark:bg-neutral-800 px-3 py-2 text-neutral-900 dark:text-neutral-100" />
  </p>
  <p>
    <label class="block font-bold mb-1" for="email">E-Mail *</label>
    <input type="email" id="email" name="email" required
      class="w-full rounded border border-neutral-300 dark:border-neutral-600 bg-neutral-50 dark:bg-neutral-800 px-3 py-2 text-neutral-900 dark:text-neutral-100" />
  </p>
  <p>
    <label class="block font-bold mb-1" for="nachricht">Nachricht *</label>
    <textarea id="nachricht" name="nachricht" rows="6" maxlength="2000" required
      class="w-full rounded border border-neutral-300 dark:border-neutral-600 bg-neutral-50 dark:bg-neutral-800 px-3 py-2 text-neutral-900 dark:text-neutral-100"></textarea>
  </p>
  <div data-netlify-recaptcha="true"></div>
  <p>
    <button type="submit"
      class="rounded bg-primary-600 px-6 py-2 font-bold text-white hover:bg-primary-500 transition-colors">Absenden</button>
  </p>
</form>
