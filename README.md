# Apprentice Progression

A record of what apprentice tailors have learned, who confirmed it, and what comes next.
Built for Gradient Pod's Webbo3 Buildathon.

**HTML and CSS only.** No JavaScript, no build step, no dependencies, no framework.
Open `index.html` in a browser and the whole thing runs.

---

## Walking through it

Everything below works right now, with no backend. Start at `index.html`.

| Route         |                                             |                                                  |
| ------------- | ------------------------------------------- | ------------------------------------------------ |
| **Public**    | `index.html`                                | Landing page                                     |
|               | `about.html`                                | How the programme works                          |
|               | `directory.html`                            | All apprentices — **filterable by stage, no JS** |
|               | `apprentice.html`                           | One apprentice's full record                     |
|               | `skill.html`                                | One confirmed skill, its evidence and sign-off   |
|               | `curriculum.html`                           | Six stages, 31 skills — **accordion, no JS**     |
|               | `workshops.html`                            | The five workshops on the register               |
| **Auth**      | `login.html`                                | Any email + password signs you in                |
|               | `signup.html`                               | Role choice, workshop picker, validation         |
|               | `forgot-password.html` → `check-email.html` | Password reset flow                              |
| **Signed in** | `dashboard.html`                            | The master's overview                            |
|               | `dashboard-apprentice.html`                 | Zainab Ibrahim's apprentice dashboard            |
|               | `confirmations.html`                        | Approval queue                                   |
|               | `record.html` → `saved.html`                | Log a milestone                                  |
|               | `settings.html`                             | **Tabbed settings, no JS**                       |
| **Utility**   | `404.html`                                  | Not found                                        |

The demo account is **Mrs. Bello, a master tailor** — she is the one with a confirmation
queue, which is why the signed-in pages are hers.

```
css/
  brand.css   tokens, reset, base type, every shared component — all pages load this
  app.css     layouts for the product pages
  home.css    landing page only
  auth.css    log in / sign up / reset
assets/img/
  hero.svg, stage-1…6.svg   drawn for this project; see note below
```

---

# Corrections — read this part

What I changed and, more importantly, **why**. The _why_ is the bit worth keeping.

## 1. Bugs that were breaking the page

**`main.html` had two `<body>` tags and a stray `</head>`:**

```html
<body></head>s      <!-- there was even a loose letter "s" rendering on the page -->
<body>
```

A browser will not tell you about this. It silently repairs the document and carries on, so
the page _looks_ fine while the DOM is not what you wrote. **Validate your HTML before you
call a page done.** All 17 pages now parse with balanced tags.

**Card 5 had an anchor closed twice with another nested inside it:**

```html
<a href="#" class="view-progress">
</a>                                            <!-- closed immediately -->
    View Progress
    <span> <a href="detail.html">→</a></span>   <!-- a link inside a link -->
</a>                                            <!-- and closed again -->
```

Nested anchors are invalid; the browser's repair moves the text _outside_ the link, so that
card's call to action was not clickable at all.

**Three links pointed at files that did not exist:** `details.html` (the file was
`detail.html`), `dashboard.html` (never written), and `<link href="brand.css">` in
`detail.html` — the file lives at `css/brand.css`, so that page loaded with **no styling
whatsoever**. Click every link on your own site before you ship it.

## 2. One design system, not three

You had **three** stylesheets defining tokens, in two different naming schemes for the same
colours:

```css
/* css/brand.css */
--color-navy: #1b3a57;
--space-3: 1rem;
/* css/main.css   */
--brand: #1b3a57;
--s2: 16px;
/* main-brand.css */
--brand: #1b3a57;
--s2: 16px; /* exact duplicate */
```

Worse, `css/main.css` contained rules using `--color-navy` and `--space-3` — tokens it never
defined. Those rules only worked on pages that happened to also load `brand.css`.

**One system now.** `css/brand.css` is the only place tokens are defined; everything else
consumes them. Change the ink colour in one line and the whole product follows. That is the
entire point of tokens, and three copies throws it away.

## 3. It was a brochure, not a product

The original was four pages: a welcome screen, a form, a card grid and a detail page. There
was no way to sign in, no account, no dashboard, no way for a master to actually _confirm_
anything — even though "confirmed by a master" is the whole idea.

A product this shape needs, at minimum: a way in (**log in / sign up / password reset**), a
home for the signed-in user (**dashboard**), the thing the product exists to do
(**a confirmation queue**), a place to manage yourself (**settings**), the reference material
(**curriculum**, **workshops**), and the pages that catch mistakes (**404**). That is what
the 17 pages are.

**Every route is reachable and every link resolves** — I check this mechanically, not by eye.

## 4. Do not ship UI that does not work

The old directory had a search box, a stage dropdown, a "6 apprentices" counter and a
"No apprentices found" panel. **None of them did anything** — they all needed the JavaScript
the brief does not allow.

This is the most important lesson here. A search box that does not search is worse than no
search box, because it makes a promise and breaks it. Ship less that works.

So the fake search is gone, and three interactions were rebuilt to **genuinely work with no
JavaScript**:

**Stage filter** — radio inputs plus the sibling combinator:

```html
<!-- the radios hold the state; they must come BEFORE what they control -->
<input class="filter-input" type="radio" name="stage" id="stage-3" />
<div class="filter-bar"><label for="stage-3">Stage 3</label></div>
<div class="roster"><article class="card" data-stage="3">…</article></div>
```

```css
#stage-3:checked ~ .roster .card {
  display: none;
}
#stage-3:checked ~ .roster .card[data-stage="3"] {
  display: flex;
}
```

`~` only looks **forward** among siblings — that is why the inputs are written first. The
radios are moved off-screen rather than `display: none`, so they stay keyboard-operable, and
their focus ring is forwarded to the visible chip.

**Settings tabs** use the same radio trick. **The curriculum accordion and both menus** use
native `<details>`/`<summary>` — real disclosure widgets with keyboard and screen-reader
behaviour built in, and not a line of script. **Reach for the platform before you reach for
a workaround.**

## 5. Responsive is not a pile of breakpoints

Three media queries to make one grid go 3 → 2 → 1 column is three numbers to maintain and
three chances to be wrong at a size you never tested.

```css
/* before */
.apprentice-grid {
  grid-template-columns: repeat(3, minmax(0, 1fr));
}
@media (max-width: 900px) {
  .apprentice-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
@media (max-width: 620px) {
  .apprentice-grid {
    grid-template-columns: 1fr;
  }
}

/* after — one rule, no breakpoints, correct at every width */
.roster {
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 17rem), 1fr));
}
```

Two details in that line that took me a moment to get right:

- **`auto-fill`, not `auto-fit`.** `auto-fit` collapses empty tracks, so when the filter
  leaves one card visible it stretches across the entire row and the illustration becomes
  enormous. I only caught this by screenshotting the _filtered_ state. **Test your states,
  not just your default view.**
- **`min(100%, 17rem)`**, not bare `17rem` — a bare floor overflows viewports narrower than
  17rem.

Your breakpoints were also in `px` while the rest of the file used `rem`. If someone raises
their default font size, `rem` breakpoints adapt and `px` ones do not. All breakpoints are
now `rem`, chosen by _where the layout actually breaks_.

There is a real mobile menu now, too — the old nav just wrapped and hoped.

## 6. Type: a font should say what the product is

Fraunces and Georgia are warm and bookish. This is a **fashion** product, so the display
face is now **Bodoni Moda** — a Didone, the genre _Vogue_ and _Harper's Bazaar_ built their
mastheads from, with the extreme thick/thin contrast that reads as fashion at a glance.
**Jost** — a geometric sans in the Futura line that fashion houses lean on — does all the
interface work.

One rule that matters: **Bodoni's hairlines disappear below about 20px.** So display type is
reserved for large sizes and the sans does every small job. Pairing a face with the wrong
size is how good typefaces get blamed for bad typography.

On sizes — you asked me not to bury the design in tiny uppercase labels, and you were right.
The old `detail.css` set every section heading to `font-size: 13px; text-transform: uppercase`.
That is decoration standing in for hierarchy, and at 13px it is genuinely hard to read.

- Body text is **17px**; the smallest text anywhere is **15px**, with a `--text-sm` floor.
- Headings use `clamp()`, scaling smoothly instead of jumping at a breakpoint.
- Hierarchy comes from **size, weight and space** — not letterspacing tricks.
- One spacing scale, used everywhere. The old files mixed `var(--s3)`, raw `px` and `rem` in
  the same rule.
- Prose is capped at `64ch`. Longer lines are measurably harder to read.

## 7. Colour: white ground, and contrast is a requirement

The background is now **white** (`--paper: #FFFFFF`), with near-black ink and a single
terracotta accent. Black buttons, hairline rules, flat fills. No gradients anywhere — I
check for that mechanically too.

The logo was sampled directly: navy `#122D4B`, amber `#DA8D1B`, and terracotta `#CD5427`.
The exact amber is decorative-only on white at **2.69:1**, and exact terracotta is reserved
for large/UI decoration at **4.29:1**. Text uses the separately named safe role tokens.
Every colour token is measured and annotated in `brand.css`:

|                        | light                            | dark             |
| ---------------------- | -------------------------------- | ---------------- |
| navy / white fill text | 13.98:1                          | 13.98:1 on navy  |
| body ink               | 17.89:1                          | 17.89:1 on white |
| muted text             | 6.85:1                           | 6.85:1 on white  |
| safe accent text       | 5.78:1                           | 5.78:1 on white  |
| logo amber / navy      | 2.69:1 on white / 5.20:1 on navy | 5.20:1           |
| logo terracotta / navy | 4.29:1 on white / 3.26:1 on navy | 3.26:1           |

**Never let colour be the only signal.** The status pills say "Confirmed", "Awaiting
confirmation", "Not started" — someone who cannot tell the hues apart still gets the
information from the words.

**A bug I introduced and had to fix, because it teaches more than the clean code does:** I
added a dark mode by swapping tokens, and `--brand` flips from dark navy to _light_ blue.
But the primary button used `--brand` as its **background** with light text — so in dark mode
it became light-on-light, about **1.4:1**. The lesson: **a token meaning "brand text colour"
cannot double as "brand fill colour"**, because the two invert in opposite directions. They
are now separate: `--fill` / `--fill-hover` / `--fill-ink`. **Name tokens for their role, not
their colour.**

## 8. Imagery should show the work, not the team

The apprentice photos were being used as the main imagery — including the hero. They are
portraits of people, not pictures of tailoring, so the site looked like a staff page rather
than a fashion product.

I drew **seven SVG illustrations** for this project (`img/`): a spool and needle, draped
cloth, shears on a cutting line, a bodice pattern with grainline and notches, a dress form,
a hanger with buttons, and a composed hero. One per stage, so the illustration on a card
tells you what that apprentice is actually learning. They are vector, a couple of kilobytes
each, sharp at any size, and each carries its own `prefers-color-scheme` block so the line
work re-colours in dark mode.

**The six `.jpeg` portraits are now unreferenced** — people are shown as initials discs
instead, which stay legible at 30px where a full-body photo crops to an unreadable smudge.
The files are still in the repo; say the word if you would rather have faces back in the
directory, or delete them.

## 9. Accessibility

- **Skip link** on every page.
- **One focus style sitewide**, via `:focus-visible`, so keyboard users get a ring and mouse
  users do not.
- The old code marked the current page with a decorative `.active` class. It now uses
  `aria-current="page"`, and the CSS styles that attribute directly:
  `.nav-desktop a[aria-current="page"]`. **One source of truth, meaningful to both screen
  readers and the stylesheet.**
- The logo `<img>` carries `alt=""` — it sits in a link that already says "Apprentice
  Progression", so describing it again is noise. **Decorative images take an empty alt, not
  a missing one.** Same for the `→` arrows (`aria-hidden="true"`).
- Progress bars carry `role="img"` and an `aria-label` reading the percentage.
- Every form control has a label — I verify this mechanically, counting both explicit
  `for=` and implicit wrapping labels.
- `.sr-only` uses `clip-path`, not `display: none` — `display: none` removes content from the
  accessibility tree entirely, which defeats the purpose.
- Touch targets are at least 44–48px. `prefers-reduced-motion` is respected.

## 10. Forms

`request.html` shipped a `<script>` block, which the brief does not allow:

```js
document.querySelector(".form").addEventListener("submit", function (event) {
  event.preventDefault();
  document.querySelector("#confirmation").hidden = false;
});
```

Gone. Forms now submit to real confirmation pages — which is what a browser does natively
and what a server-backed form would do anyway.

Also added: `autocomplete` so browsers can fill fields in, `inputmode="numeric"` for number
pads on phones, `min`/`max`, `<datalist>` for the workshop picker, fields grouped into
labelled `<fieldset>`s, and _optional_ marked instead of _required_ — on these forms nearly
everything is required, so marking the exception is less visual noise.

Validation uses **`:user-invalid`, not `:invalid`**. `:invalid` turns a required field red
the instant the page loads — telling someone off for something they have not done yet.

**Two CSS traps I hit while building this**, both worth knowing:

_A `<legend>` is laid out inside its fieldset's top border_, so a `border-top` on the
fieldset draws straight through the heading text. Float it to escape the notch:

```css
fieldset > legend {
  float: left;
  width: 100%;
  border-bottom: 1px solid var(--line);
}
fieldset > legend + * {
  clear: both;
} /* a float must be cleared */
```

_A stacking margin fires inside a grid._ I had `.field + .field { margin-top: 1.5rem }` for
vertical stacking, but in a two-column `.field-row` that margin pushed the second column
down and the two inputs stopped lining up:

```css
.field-row > .field + .field {
  margin-top: 0;
}
```

## 11. Naming and dead code

`main.html` tells you nothing about what a page contains. Files are named for their content
now, and `index.html` exists — without it, a web server shows a **file listing** instead of
your site. Renames were done with `git mv`, so history follows the files.

`css/main.css` was 792 lines with whole blocks duplicated — `.logo` appeared twice in the
same file, and `.site-header` was defined identically in two stylesheets. **Duplicated CSS
does not stay duplicated; it drifts, and then you have two truths.**

I also deleted every rule nothing uses. I ended up removing five of my own (`.data-table`,
`.table-scroll`, two `.notice` variants, `.split--wide-aside`) — writing dead CSS while
lecturing you about dead CSS would have been a poor look.

## 12. Performance

- **Every `<img>` has `width` and `height`.** Without them the browser cannot reserve space
  and the page jumps as images load. Card media also sits in an `aspect-ratio` box.
- Below-the-fold images get `loading="lazy"`; the hero gets `fetchpriority="high"`.
- **Font faces: eight before, five now.** The old `main.html` requested Fraunces at three
  weights plus an italic and Public Sans at four. I audited which weights the CSS actually
  uses — Bodoni at 600 plus one italic for the pull quote, Jost at 400/500/600 — and
  requested exactly those. Every unused weight is a separate file the browser downloads
  before it can paint text.
- The illustrations are SVG — a couple of KB each instead of the 40–100 KB JPEGs.

---

## Habits worth taking to the next project

1. **Validate your HTML.** The browser hides your mistakes; a validator does not.
2. **Click every link.** Three of yours were broken.
3. **Never ship a control that does nothing.** Build it, or leave it out.
4. **Define a token once.** Three copies is three values waiting to diverge.
5. **Check contrast with a tool.** Yours looked fine and still failed AA.
6. **Look at every state.** The `auto-fit` bug was invisible until I rendered the filtered view.
7. **Let the platform do the work** — `<details>` instead of a script, `auto-fill` instead of
   breakpoints, form submission instead of `preventDefault`, `aria-current` instead of a class.
8. **Name things for their role, not their appearance.** `--fill-ink` survives a theme swap;
   `--navy` does not.

Your original had real instincts in it: the six-stage model, the progress meter, the
confirmed-by-a-master idea, and the decision to use CSS custom properties at all. Those were
the right calls, and they are all still here. What needed work was the discipline underneath
them — and the ambition about what the thing actually had to _do_.
