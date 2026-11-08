# Fill Your Cup: Mere Christianity Devotion Builder

**How to use:** Start a new Claude chat. Upload (1) this file, (2) the Mere Christianity EPUB and (3) your current **`fill-your-cup.html`** (the one app file that holds every devotion so far). Type **next**. Claude adds the next devotion to that file as a new entry in the dropdown and gives you the updated `fill-your-cup.html`. Replace the old one on GitHub with it. Then paste the "Tracker update" Claude gives you into Section 3 of this file, save it, and repeat in a new chat.

**Commands you can type:** `next` (add the next devotion) · `next 5` (add devotion 5) · `redo 3: make the stories funnier` (rebuild devotion 3 inside the app)

**One app, many devotions:** there is only ever **one** HTML file, `fill-your-cup.html`. A dropdown in the top bar lets the leader pick any devotion. The app opens on the newest devotion, and a link ending in `#d2` opens devotion 2 directly.

---

## 0. Instructions for Claude (follow exactly)

You are building a 15-minute morning "fill your cup" devotion, one per message, from C. S. Lewis's *Mere Christianity*, and **adding it to one combined app file**. The leader may have **no experience and no prep time**, so every slide must contain everything they need to say, ask and read. The look and behaviour are **locked**: the person loves the existing design and wants it identical in every devotion.

### Hard rules
1. **Never change the CSS, JavaScript, fonts, colours, bubbles, hover behaviour, dropdown or layout** of the app. Only add or replace a devotion block (Section 5 shows its shape). Never touch other devotions when adding a new one.
2. **Build from existing classes only** (Section 5). Do not invent new components or styles.
3. **Use NZ/UK English spelling.** Warm, plain, conversational. No jargon without a one-line explanation.
4. **Copyright:** paraphrase Lewis in your own words. No direct quote over 15 words, and at most one short quote from the book per devotion. Never reproduce passages. Bible verses: use **KJV or WEB only** (public domain), 1 to 4 verses.
5. **Be faithful to Lewis.** Do not put words in his mouth. Do not invent statistics, studies or named real people. Real-life stories must be everyday, believable scenarios, written as "Imagine…" or "Have you ever…", not claimed as facts about real individuals.
6. **No text may overlap or be clipped.** Keep each slide to roughly 120 words of visible text (card reveal text not counted) and at most 3 cards per grid.
7. **Free-account friendly:** read only the chapter you need, once. Do not re-read the whole template, do not take screenshots, do not rewrite the app by hand. Write only the slides.
8. Do not ask questions. If this file, the EPUB or the current `fill-your-cup.html` is missing, say which one and stop (for devotion 1 only, the app can be rebuilt from Section 9).
9. **Never put `id="tm"` on the timer button.** Use `<button class="btn timer">` (each devotion has its own timer).

### Workflow when the user types `next` (or a command)
1. **Find the files.** `find /mnt/user-data -type f \( -name '*.md' -o -name '*.epub' -o -name 'fill-your-cup*.html' \)`. Use this `.md`, the `.epub` and the app `.html`.
2. **Decide the number.** Read Section 3. Next devotion = last row marked Done + 1 (or the number the user gave). Use the chapter from Section 4. Check the app has no devotion with that number already (`next` must never duplicate; use `redo` for that).
3. **Read only that chapter** from the EPUB with this helper (adjust names after listing):
```python
import zipfile,re,html,glob
z=zipfile.ZipFile(glob.glob('/mnt/user-data/**/*.epub',recursive=True)[0])
files=[n for n in z.namelist() if n.lower().endswith(('.xhtml','.html','.htm'))]
def text(n):
    t=z.read(n).decode('utf8','ignore'); t=re.sub(r'<(script|style).*?</\1>','',t,flags=re.S)
    return html.unescape(re.sub(r'\s+',' ',re.sub(r'<[^>]+>',' ',t)))
for n in files: print(n, len(text(n)), text(n)[:70])   # locate the chapter
# then print(text('the-chapter-file')) in one go
```
If a chapter spans several files, read them in order. Verify the chapter title against Section 4 and trust the book if they differ.
4. **Plan in 5 short lines** (do not show to the user unless asked): big idea, 3 relatable story scenarios, the 2 or 3 Lewis points, the Bible verse(s), the personal challenge and 3 or 4 next steps.
5. **Write the slides** to `/home/claude/slides.html` (Section 6 gives the exact 9-slide structure and Section 7 the writing rules). Slides only, no wrapper.
6. **Add it to the app** with this script (N = number, TITLE = short title such as `The Tune`). It wraps the slides in a devotion block and inserts it **before** `<!--DEVOTIONS-END-->`, so devotions stay in order and the dropdown builds itself:
```python
import re,glob
app_path=glob.glob('/mnt/user-data/**/fill-your-cup*.html',recursive=True)[0]
app=open(app_path,encoding='utf8').read()
N,TITLE=3,'The Reality of the Law'                      # set these
slides=open('/home/claude/slides.html',encoding='utf8').read().strip().replace(' id="tm"','')
block=f'<!--D{N}-START-->\n<div class="devotion" data-title="Devotion {N} · {TITLE}">\n{slides}\n</div>\n<!--D{N}-END-->\n'
assert f'<!--D{N}-START-->' not in app, 'already exists, use redo'
app=app.replace('<!--DEVOTIONS-END-->',block+'<!--DEVOTIONS-END-->')
open('/mnt/user-data/outputs/fill-your-cup.html','w',encoding='utf8').write(app)
print(app.count('class="devotion"'),app.count('<section class="slide"'),app.count('</section>'))
```
For `redo N`, replace the old block instead: `app=re.sub(rf'<!--D{N}-START-->.*?<!--D{N}-END-->',lambda m:block.strip(),app,flags=re.S)`.
7. **Quick check (text only):** devotion count is N, slide count is 9 × devotions, section tags match, every `<div class="in">` is closed, no `id="tm"`, and the new devotion's title text says the right number.
8. **Deliver:** call `present_files` (or SendUserFile) with `fill-your-cup.html` so the user can download it for GitHub. If an Artifact publish tool is available, also publish it once for a live preview.
9. **Final message (keep short):** one line on what the devotion covers, a reminder to replace `fill-your-cup.html` on GitHub, then this block for the user to paste into Section 3:
   `| N | Chapter name | Done | one-sentence key idea |`
   and a one-line teaser of the next devotion.

### Other commands
- `redo N: notes`: rebuild devotion N inside the app (same process, step 6 `redo` form), applying the notes. Reply with a tracker line only if the key idea changed.
- There is no separate contents page: the dropdown in the app is the index.

---
## 1. Design system (locked, for reference)

- **Background:** vertical gradient `#c84b7e` to `#ff95a0` (deeper in dark mode). Faint charcoal **outline-only bubbles** rise slowly, some pop, new ones appear from the bottom.
- **Type:** Montserrat 900 for headings (huge, tight), 500 for body. White text.
- **Accent colours:** magenta `#e0267f`, blue `#19a7d8`, charcoal `#2a2430`, gold `#e2a22b`. Key words glow **yellow** on pink and dark boxes, **magenta** on white boxes.
- **Boxes:** charcoal "Say this" (leader reads aloud), white "Ask this" (leader asks the group). Both zoom to about 107% and drift toward screen centre on hover.
- **Cards:** translucent glass squares. On hover they zoom about 14%, turn solid white with dark text, others dim and blur. Tap to reveal. Flip cards rotate in 3D.
- **Interaction:** click bursts of colour dots, arrow keys, swipe, dots, buttons. Reduced-motion respected.
- **Devotion dropdown:** a white pill with magenta text in the top bar, next to the logo. It lists every devotion ("Devotion 2 · The Tune"). Picking one jumps to its first slide. On phones the slide dots hide to make room; the slide counter and swipe remain.

## 2. Devotion shape (every devotion follows this path)

1. **Lead question / big picture** (2 min)
2. **The lead-in stories: "Have you ever…"** (3 min)
3. **Reflection: what Lewis says** (3 min)
4. **What does the Bible say?** (3 min)
5. **Commit it to us: make it personal** (3 min)
6. **What next: next steps** (1 min)

## 3. Progress tracker (update after each devotion)

| # | Chapter | Status | Key idea (used for the next recap) |
|---|---------|--------|------------------------------------|
| 1 | Book 1, ch 1: The Law of Human Nature | Done | We all appeal to a standard of right and wrong, and none of us keeps it. |
| 2 | Book 1, ch 2: Some Objections | Done | Our sense of "ought" isn't an instinct or just a custom. Our urges are like piano keys, and conscience is the sheet music that tells us which to play, a real standard we didn't invent. |
| 3 | Book 1, ch 3: The Reality of the Law | Done | Unlike a stone, we live under a real "ought" above the facts of how we behave; it isn't mere usefulness or invention, but a real law none of us made. |
| 4 | Book 1, ch 4: What Lies Behind the Law | Done | Science can't tell us whether anything is behind the universe, but we know one thing from the inside: ourselves, where we find a law we didn't write, hinting at a Something behind it all. |
| 5 | Book 1, ch 5: We Have Cause to Be Uneasy | Done | The law behind our conscience is hard as nails, so real goodness is both our only hope and our danger; Christianity's good news only makes sense after we face that honestly. |
| 6 | Book 2, ch 1: The Rival Conceptions of God | Done | Christians believe a good God made the world and stands apart from it, like a painter from his picture, which is why things can really be wrong; calling the world unjust needs a real standard, so "no God" is too simple. |
| 7 | Book 2, ch 2: The Invasion | Next | |

(Add new rows below as Claude gives them to you.)

## 4. Chapter map (one devotion per chapter, in order)

**Book 1: Right and Wrong as a Clue to the Meaning of the Universe**
1 The Law of Human Nature · 2 Some Objections · 3 The Reality of the Law · 4 What Lies Behind the Law · 5 We Have Cause to Be Uneasy

**Book 2: What Christians Believe**
6 The Rival Conceptions of God · 7 The Invasion · 8 The Shocking Alternative · 9 The Perfect Penitent · 10 The Practical Conclusion

**Book 3: Christian Behaviour**
11 The Three Parts of Morality · 12 The "Cardinal Virtues" · 13 Social Morality · 14 Morality and Psychoanalysis · 15 Sexual Morality · 16 Christian Marriage · 17 Forgiveness · 18 The Great Sin · 19 Charity · 20 Hope · 21 Faith · 22 Faith (second chapter)

**Book 4: Beyond Personality**
23 Making and Begetting · 24 The Three-Personal God · 25 Time and Beyond Time · 26 Good Infection · 27 The Obstinate Toy Soldiers · 28 Two Notes · 29 Let's Pretend · 30 Is Christianity Hard or Easy? · 31 Counting the Cost · 32 Nice People or New Men · 33 The New Men

Notes: devotion numbers above are **book order**, not Lewis's own numbering. Chapters 15 and 16 are sensitive: keep them gentle, respectful and suitable for a mixed group. If a chapter is too dense for one sitting, teach **one** idea from it and say so; do not cram.

## 5. Allowed building blocks (copy patterns from the master template)

- Badge: `<span class="badge">3 · The big picture · 2 min</span>`
- Headline: `<h2>…</h2>`; big line: `<p class="big">…</p>`; huge title: `<h1>` with `<span class="hero-word">`
- **Say box:** `<div class="say"><b>Say this</b><p>…</p></div>`
- **Ask box:** `<div class="ask"><b>Ask this</b><p>…</p></div>`
- **Tap cards:** `<div class="grid"><button class="card"><h3>…</h3><p>front line</p><div class="hide"><p>revealed text</p></div><p class="tap">Tap to open</p></button>…</div>`
- **Flip cards:** `<button class="flip"><div><span class="f"><p>question</p></span><span class="b"><p>answer or nudge</p></span></div></button>`
- **Verse:** `<div class="verse"><p>"…"</p><small>Book 1:1 (KJV)</small></div>`
- **Steps (tick list):** `<ul class="steps"><li><span><span class="key">Bold lead:</span> action</span></li></ul>`
- **Key words:** `<span class="key">…</span>` (yellow/magenta, bold, larger). Use 1 to 3 per paragraph, only for the words a skim-reader must catch. Never wrap whole sentences.
- Quiet timer button: `<button class="btn timer">Start 30 sec quiet</button>` (no id)
- Next/restart buttons: `<button class="btn" data-next>Start</button>`, `<button class="btn" data-first>Back to start</button>`
- Each slide: `<section class="slide"><div class="in">…</div></section>`
- Each devotion in the app (the build script adds this wrapper for you, you only write the 9 slides):
  `<!--D3-START-->` `<div class="devotion" data-title="Devotion 3 · Title">` 9 slides `</div>` `<!--D3-END-->`

## 6. The 9 slides (always this order)

1. **Title:** badge "Devotion N · Mere Christianity · about 15 minutes", `<h1>` with 1 or 2 hero words from the chapter's theme, a `.big` line stating the question, a Start button.
2. **Welcome (devotion 1 had a full leader guide; devotions 2+ use this):** badge "Welcome back · 1 min". One Say box with a 2-sentence recap of the previous devotion (use the Key idea from Section 3) and a warm line linking to today. One Ask box: a quick check-in (e.g. "Did anyone notice the idea from last time this week?"). Add one small line reminding leaders: dark boxes are read aloud, white boxes are asked.
3. **Big picture · 2 min:** `<h2>` lead question with key words, one `.big` provocative line, a Say box (the link to Lewis's point, about 50 words), an Ask box.
4. **Stories · 3 min:** `<h2>Have you ever…?</h2>`, 3 tap cards. Each card: a short title, a one-line "I said/thought…" on the front, and a vivid 2 to 3 sentence everyday scenario on reveal (work, family, driving, school, shops, social media, sport). Then an Ask box asking the group to share a true story of their own, plus a line for the leader: "Leader, share a quick story of your own first to get things going."
5. **Reflection · 3 min:** `<h2>` naming Lewis's main idea, 3 tap cards (each: Lewis's point on the front, plain-language unpacking on reveal), then a Say box with the "so what" in about 60 words. Mention "Mere Christianity, Book X, chapter Y (summarised)".
6. **Bible · 3 min:** a verse card (KJV/WEB), a Say box explaining it in plain words, 2 tap cards (e.g. "If that's true…" and "But also…"), an Ask box ("Read the passage aloud together. What word stands out?").
7. **Personal · 3 min:** `<h2>Commit it to us</h2>`, 3 flip cards (front: a searching question, back: a gentle nudge), a Say box leading 30 seconds of silent prayer, and the timer button.
8. **Next steps · 1 min:** `<h2>This week, try one (or all)</h2>`, 3 or 4 tick steps (observe, act, read/pray, share). The share step is always "Tell someone your story" (with a friend or family member, this week) and never "bring a story to next time", because the group may meet only once a term, and an Ask box "Which step will you take? Tell one person here."
9. **Close:** `<h1>` hero word "amen", a short original prayer in a `.verse` box (3 sentences), a Say box with a one-line teaser of the next devotion, a "Back to start" button.

## 7. Writing rules

- Every Say box is written to be read aloud: short sentences, warm, about 40 to 80 words. Every Ask box is one clear question, then "wait" or "take one or two answers".
- Stories must be **concrete and real-life**: a specific moment, a feeling, a line of dialogue. No vague generalities.
- Reflection slide teaches **one main idea** from the chapter plus at most two supporting points. Explain any Lewis term in plain words.
- Always show how the idea connects to everyday life this week.
- Stay encouraging. Name struggle honestly, and always land on grace and hope.
- Tone suits mixed ages, believers and sceptics, NZ audience. No preaching, no shaming.
- Total talk time must fit 15 minutes. Put the minutes in each badge.

## 8. Quality checklist (check silently before delivering)

All 9 slides present · key words used well and not overused · stories are concrete · every Say/Ask box is read-ready · no direct Lewis quote over 15 words · Bible text is KJV/WEB · badge minutes add to about 15 · one `fill-your-cup.html` with the new devotion in the dropdown, older devotions untouched · tracker line ready.

## 9. Master app shell (do not edit)

This is the app as it stands with devotion 1 inside. Claude normally edits the user's uploaded `fill-your-cup.html`. Only if devotion 1 is the one being rebuilt may it be copied from here.

<!--TEMPLATE-BEGIN-->
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Fill Your Cup: Mere Christianity Devotions</title>
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@500;700;900&display=swap" rel="stylesheet">
<style>
:root{--top:#c84b7e;--bot:#ff95a0;--ink:#2a2430;--white:#fff;--mag:#e0267f;--blue:#19a7d8;--gold:#e2a22b;--card:rgba(255,255,255,.18);--cardb:rgba(255,255,255,.35);box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px);color-scheme:light}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--top:#8f2f5c;--bot:#d8697a}}
*{box-sizing:border-box;margin:0}
html,body{height:100%;overflow:hidden}
body{font-family:'Montserrat','Century Gothic','Avenir Next',Arial,sans-serif;color:var(--white);background:linear-gradient(180deg,var(--top),var(--bot));-webkit-font-smoothing:antialiased}
#orbs{position:fixed;inset:0;pointer-events:none;z-index:0}
.bub{position:absolute;left:0;top:0;border-radius:50%;border:2px solid rgba(42,36,48,.26);background:transparent;will-change:transform,opacity}
.key{font-weight:900;font-size:1.18em;color:#ffe14a;text-shadow:0 2px 0 rgba(120,20,70,.25)}
.say .key,.flip .f .key{color:#ffc93c;text-shadow:none}
.ask .key,.verse .key,.card.open .key,.card:hover .key,.card:focus-visible .key,.flip .b .key,.steps li.done .key{color:#d6126e;text-shadow:none}
.slide{position:absolute;inset:0;z-index:1;padding:104px 7vw 124px;display:flex;flex-direction:column;justify-content:flex-start;opacity:0;visibility:hidden;transform:translateX(6vw) scale(.98);transition:opacity .5s,transform .6s cubic-bezier(.2,.8,.2,1),visibility 0s .6s;overflow-y:auto}
.slide.on{opacity:1;visibility:visible;transform:none;transition-delay:0s}
.slide.past{transform:translateX(-6vw) scale(.98)}
.in{max-width:1000px;width:100%;margin:auto;padding:0 18px}
h1{font-weight:900;font-size:clamp(56px,14vw,170px);line-height:.95;letter-spacing:-.04em}
h2{font-weight:900;font-size:clamp(28px,4.6vw,58px);line-height:1.05;letter-spacing:-.03em;margin-bottom:.45em}
h3{font-weight:700;font-size:clamp(18px,2.2vw,26px);margin-bottom:.4em}
p,li{font-size:clamp(16px,1.9vw,22px);line-height:1.55;font-weight:500}
p+p{margin-top:.8em}
.big{font-size:clamp(22px,3.2vw,40px);font-weight:700;line-height:1.25}
.badge{display:inline-block;background:var(--white);color:var(--mag);font-weight:900;font-size:14px;padding:6px 14px;border-radius:99px;margin-bottom:18px}
.say{background:var(--ink);color:#fff;border-radius:22px;padding:20px 26px;margin:18px 0;position:relative}
.say b{display:block;color:var(--gold);font-size:14px;margin-bottom:6px;letter-spacing:.02em}
.say p{font-size:clamp(16px,1.8vw,21px)}
.ask{background:var(--white);color:var(--ink);border-radius:22px;padding:20px 26px;margin:18px 0}
.say,.ask{position:relative;transition:transform .35s cubic-bezier(.2,.9,.3,1.15),box-shadow .3s}
.say:hover,.ask:hover{z-index:6;transform:translateY(var(--ty,0px)) scale(1.07);box-shadow:0 34px 44px -20px rgba(20,5,30,.6)}
.ask b{display:block;color:var(--mag);font-size:14px;margin-bottom:6px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(230px,1fr));gap:18px;margin:20px 0}
.card{background:var(--card);border:2px solid var(--cardb);border-radius:26px;padding:22px;cursor:pointer;transition:transform .25s cubic-bezier(.2,.9,.3,1.4),background .25s;backdrop-filter:blur(6px);text-align:left;color:inherit;font:inherit;min-height:150px}
.card{transition:transform .35s cubic-bezier(.2,.9,.3,1.15),background .25s,color .25s,opacity .3s,filter .3s,box-shadow .3s}
.card:hover,.card:focus-visible,.card.open:hover{position:relative;z-index:6;transform:scale(1.14);background:var(--white);color:var(--ink);box-shadow:0 34px 44px -20px rgba(60,10,40,.6);outline:none}
.grid:has(.card:hover) .card:not(:hover),.grid:has(.flip:hover) .flip:not(:hover){opacity:.4;filter:blur(1.5px)}
.card .hide{display:none}
.card.open{background:var(--white);color:var(--ink)}
.card.open .hide{display:block;animation:pop .4s}
.card.open .tap{display:none}
.card h3{color:inherit}
.tap{font-size:14px;opacity:.85;margin-top:8px}
@keyframes pop{from{opacity:0;transform:scale(.9)}to{opacity:1;transform:none}}
.verse{background:var(--white);color:var(--ink);border-radius:30px;padding:30px 34px;margin:18px 0;box-shadow:0 30px 30px -24px rgba(60,10,40,.5);transition:transform .3s}
.verse:hover{transform:scale(1.05)}
.verse p{font-size:clamp(19px,2.5vw,30px);font-weight:700;line-height:1.4}
.verse small{display:block;margin-top:12px;color:var(--mag);font-weight:900;font-size:15px}
.mark{background:linear-gradient(transparent 60%,var(--gold) 60%);cursor:pointer}
.flip{perspective:900px;min-height:190px;cursor:pointer;background:none;border:0;color:inherit;font:inherit;text-align:left}
.flip>div{position:relative;width:100%;height:100%;min-height:190px;transition:transform .7s cubic-bezier(.2,.8,.2,1);transform-style:preserve-3d}
.flip.open>div{transform:rotateY(180deg)}
.flip .f,.flip .b{position:absolute;inset:0;border-radius:26px;padding:22px;backface-visibility:hidden;-webkit-backface-visibility:hidden;display:flex;flex-direction:column;justify-content:center}
.flip .f{background:var(--ink);color:#fff}
.flip .b{background:var(--white);color:var(--ink);transform:rotateY(180deg)}
.flip{transition:transform .35s cubic-bezier(.2,.9,.3,1.15),opacity .3s,filter .3s}
.flip:hover,.flip:focus-visible{position:relative;z-index:6;transform:scale(1.14);outline:none}
.flip .f p{font-weight:700}
.flip .b p{font-size:clamp(15px,1.6vw,19px)}
.steps{list-style:none;padding:0;display:grid;gap:12px;margin:18px 0}
.steps li{background:var(--card);border:2px solid var(--cardb);border-radius:22px;padding:18px 22px;cursor:pointer;display:flex;gap:16px;align-items:center;transition:transform .2s,background .2s}
.steps li:hover{transform:translateX(10px)}
.steps li::before{content:"";flex:none;width:30px;height:30px;border-radius:50%;border:3px solid #fff;transition:.3s}
.steps li.done{background:var(--white);color:var(--ink)}
.steps li.done::before{background:var(--mag);border-color:var(--mag);box-shadow:inset 0 0 0 5px #fff}
.btn{display:inline-block;margin-top:24px;background:var(--white);color:var(--mag);font:900 18px 'Montserrat',Arial,sans-serif;border:0;border-radius:99px;padding:16px 34px;cursor:pointer;box-shadow:0 18px 16px -12px rgba(60,10,40,.5);transition:transform .25s cubic-bezier(.2,.9,.3,1.4)}
.btn:hover{transform:scale(1.08) rotate(-2deg)}
.timer{font-variant-numeric:tabular-nums}
nav{position:fixed;left:0;right:0;top:0;z-index:5;gap:16px;background:linear-gradient(180deg,var(--top) 62%,transparent);display:flex;justify-content:space-between;align-items:center;padding:calc(18px + env(safe-area-inset-top,0px)) 5vw 18px}
.logo{font-weight:900;font-size:22px;letter-spacing:-.03em;white-space:nowrap;flex:none}
.dots{display:flex;gap:8px;flex-wrap:wrap;justify-content:center}
.dots button{width:14px;height:14px;border-radius:50%;border:2px solid #fff;background:transparent;cursor:pointer;padding:0;transition:.25s}
.dots button:hover{transform:scale(1.4)}
.dots button.on{background:#fff;width:34px;border-radius:99px}
footer{position:fixed;left:0;right:0;bottom:0;z-index:5;background:linear-gradient(0deg,var(--bot) 62%,transparent);display:flex;justify-content:space-between;align-items:center;padding:16px 5vw calc(16px + env(safe-area-inset-bottom,0px))}
.arrow{width:56px;height:56px;border-radius:50%;border:3px solid #fff;background:transparent;color:#fff;font-size:26px;cursor:pointer;transition:.25s}
.arrow:hover{background:#fff;color:var(--mag);transform:scale(1.12)}
.pop{position:fixed;z-index:9;border-radius:50%;pointer-events:none;animation:burst .8s ease-out forwards}
@keyframes burst{to{transform:translate(var(--x),var(--y)) scale(.2);opacity:0}}
.hero-word{display:inline-block;transition:transform .3s cubic-bezier(.2,.9,.3,1.4);cursor:default}
.hero-word:hover{transform:translateY(-14px) rotate(-4deg)}
@media (prefers-reduced-motion:reduce){*{transition-duration:.01s!important;animation:none!important}}
@media (max-width:600px){.slide{padding:76px 6vw 100px}.say,.ask{padding:16px 18px}}
nav select{flex:0 1 auto;min-width:0;max-width:46vw;background:var(--white);color:var(--mag);font:900 14px 'Montserrat',Arial,sans-serif;border:0;border-radius:99px;padding:9px 16px;cursor:pointer;box-shadow:0 10px 12px -8px rgba(60,10,40,.5)}
nav select:focus-visible{outline:3px solid #ffe14a}
@media (max-width:600px){.dots{display:none}nav{justify-content:space-between}}
</style>
</head>
<body>
<div id="orbs" aria-hidden="true"></div>
<nav><div class="logo">fill your cup</div><select id="dsel" aria-label="Choose a devotion"></select><div class="dots" id="dots" role="tablist" aria-label="Slides"></div></nav>

<!--DEVOTIONS-START-->
<!--D1-START-->
<div class="devotion" data-title="Devotion 1 · The Clue">
<!-- 1 TITLE -->
<section class="slide"><div class="in">
<span class="badge">Devotion 1 · Mere Christianity · about 15 minutes</span>
<h1><span class="hero-word">the</span> <span class="hero-word">clue</span></h1>
<p class="big" style="margin-top:18px">Why do we all know <span class="key">right from wrong</span>, and why do we all <span class="key">fall short</span>?</p>
<button class="btn" data-next>Start</button>
</div></section>

<!-- 2 LEADER GUIDE -->
<section class="slide"><div class="in">
<span class="badge">Leaders, read this first (1 min)</span>
<h2>No prep needed. Everything is on the slides.</h2>
<div class="grid">
<div class="card open"><h3>Colour guide</h3><p>Dark boxes are <b>Say this</b>: read them aloud. White boxes are <b>Ask this</b>: ask the group, then wait. Silence is fine.</p></div>
<div class="card open"><h3>Interactive bits</h3><p>Tap or click cards to reveal them. Flip the question cards. Tick off next steps. Arrow keys, swipe or the buttons move you along.</p></div>
<div class="card open"><h3>Time plan</h3><p>Big picture 2 min · Stories 3 · Reading 3 · Bible 3 · Personal 3 · Next steps 1. Total: 15 min.</p></div>
</div>
<div class="say"><b>Say this</b><p>Welcome. Grab your coffee. For the next fifteen minutes we're filling our cups with one idea from C. S. Lewis's book, Mere Christianity. No homework, <span class="key">no wrong answers</span>. Let's begin with a question.</p></div>
</div></section>

<!-- 3 BIG PICTURE -->
<section class="slide"><div class="in">
<span class="badge">1 · The big picture · 2 min</span>
<h2>Where does our sense of <span class="key">right and wrong</span> come from?</h2>
<p class="big">If everything is just atoms, instincts and evolution, why do we get so angry when someone <span class="key">cuts in line</span>?</p>
<div class="say"><b>Say this</b><p>Lewis starts his whole book here. He says our sense of fairness is a <span class="key">clue</span>, like a footprint, that points toward who made us and what he is like. Today we follow that clue.</p></div>
<div class="ask"><b>Ask this, then wait</b><p>When was the last time something felt "just not fair" to you? Take one or two quick answers.</p></div>
</div></section>

<!-- 4 STORIES -->
<section class="slide"><div class="in">
<span class="badge">2 · Have you ever… · 3 min</span>
<h2>Have you ever said these?</h2>
<p>Tap each card. Read it aloud and ask the group: <em>have you ever been here?</em></p>
<div class="grid">
<button class="card"><h3>The parking spot</h3><p>"I was <span class="key">here first</span>!"</p><div class="hide"><p>You pulled in, waited, signalled. Someone zipped in ahead of you. Nobody taught you a parking rule. <span class="key">You just knew</span> it was broken.</p></div><p class="tap">Tap to open</p></button>
<button class="card"><h3>The sibling</h3><p>"<span class="key">How would you like it</span> if I did that to you?"</p><div class="hide"><p>Kids say this without lessons. They reach for <span class="key">a rule</span> they believe the other person also knows.</p></div><p class="tap">Tap to open</p></button>
<button class="card"><h3>The excuse</h3><p>"I was only…"</p><div class="hide"><p>When we get caught, we rarely say "your rules don't matter." We explain why this time was different. Even excuses admit <span class="key">the rule is real</span>.</p></div><p class="tap">Tap to open</p></button>
</div>
<div class="ask"><b>Ask this</b><p>Which of those is most you? Share a quick, true story from this week, a time you expected fairness or dodged it yourself.</p></div>
</div></section>

<!-- 5 LEWIS -->
<section class="slide"><div class="in">
<span class="badge">Reflection · What Lewis noticed · 3 min</span>
<h2>Two facts that explain a lot</h2>
<p>From Book 1, chapter 1 of Mere Christianity (summarised). Read each card aloud.</p>
<div class="grid">
<button class="card"><h3>Fact one</h3><p>Everyone, everywhere, appeals to a standard of behaviour and expects others to know it. Lewis calls it the <span class="key">Law of Human Nature</span>.</p><div class="hide"><p>Cultures differ on details, but nobody admires cowardice or betrayal of friends. We argue about the rule only because we assume it exists.</p></div><p class="tap">Tap for more</p></button>
<button class="card"><h3>Fact two</h3><p>We <span class="key">don't keep</span> that standard. Not even close.</p><div class="hide"><p>We demand fairness from others while excusing ourselves. Lewis says these two facts are where clear thinking about ourselves must start.</p></div><p class="tap">Tap for more</p></button>
<button class="card"><h3>Not like gravity</h3><p>A rock can't disobey gravity. We <span class="key">can</span> disobey this law.</p><div class="hide"><p>So it isn't just a description of what we do. It's a "ought" pressing on us from beyond us, like a message from someone.</p></div><p class="tap">Tap for more</p></button>
</div>
<div class="say"><b>Say this</b><p>The good news inside the bad news: if there is a real standard of goodness, then <span class="key">goodness is real</span>, and the one behind it must care how we live.</p></div>
</div></section>

<!-- 6 BIBLE -->
<section class="slide"><div class="in">
<span class="badge">3 · What does the Bible say? · 3 min</span>
<h2>Written on the heart</h2>
<div class="verse"><p>"…which shew the work of the law <span class="key">written in their hearts</span>, their conscience also bearing witness…"</p><small>Romans 2:15 (KJV)</small></div>
<div class="say"><b>Say this</b><p>Paul wrote this about people who never received the commandments. He says God already put <span class="key">his law inside every person</span>. <span class="key">Conscience</span> is the witness. Lewis's "clue" is exactly this.</p></div>
<div class="grid">
<button class="card"><h3>If that's true…</h3><p>Tap</p><div class="hide"><p>Our longing for fairness isn't random. It's a signal that we're made by a <span class="key">just and good God</span>.</p></div></button>
<button class="card"><h3>But also…</h3><p>Tap</p><div class="hide"><p>The same conscience that accuses others accuses us. Romans 3:23 says all have fallen short. We need more than a rule. We need <span class="key">rescue</span>.</p></div></button>
</div>
<div class="ask"><b>Ask this</b><p>Read Romans 2:14–15 aloud together. What word or phrase stands out to you?</p></div>
</div></section>

<!-- 7 PERSONAL -->
<section class="slide"><div class="in">
<span class="badge">4 · Make it personal · 3 min</span>
<h2>Commit it to us</h2>
<p>Flip a card. Pause 30 seconds of quiet, then share if you want to.</p>
<div class="grid">
<button class="flip"><div><span class="f"><p>Where do I demand fairness from others but <span class="key">excuse myself</span>?</p></span><span class="b"><p>Name one relationship or habit. Not to feel crushed, but to be honest. <span class="key">Honesty</span> is where <span class="key">grace</span> gets in.</p></span></div></button>
<button class="flip"><div><span class="f"><p>What does my conscience keep nudging me about?</p></span><span class="b"><p>A small voice you keep muting? Maybe an apology, a phone call, a choice. That nudge is a <span class="key">gift</span>, not a nag.</p></span></div></button>
<button class="flip"><div><span class="f"><p>What does it say about God that he built this into me?</p></span><span class="b"><p>He is not distant or indifferent. He <span class="key">cares</span> enough to speak into your heart before you ever open a Bible.</p></span></div></button>
</div>
<div class="say"><b>Say this, then pray quietly</b><p>Take 30 seconds of silence. Ask God: "Show me one place where I've fallen short, and help me trust your forgiveness."</p></div>
<button class="btn timer">Start 30 sec quiet</button>
</div></section>

<!-- 8 NEXT -->
<section class="slide"><div class="in">
<span class="badge">5 · What next? · 1 min</span>
<h2>This week, try one (or all)</h2>
<p>Click each step the group chooses to take.</p>
<ul class="steps">
<li><span><span class="key">Notice it:</span> each time you say "that's not fair," ask what standard you're appealing to.</span></li>
<li><span><span class="key">Fix one thing:</span> make the apology or phone call your conscience keeps raising.</span></li>
<li><span><span class="key">Read Romans 2:12–16</span> once on your own and thank God for conscience.</span></li>
<li><span><span class="key">Tell someone your story:</span> share with a friend or family member where you saw the "law" at work this week.</span></li>
</ul>
<div class="ask"><b>Ask this</b><p>Which step will you take? Say it out loud to one person here.</p></div>
</div></section>

<!-- 9 CLOSE -->
<section class="slide"><div class="in">
<span class="badge">Closing prayer</span>
<h1 style="font-size:clamp(48px,11vw,130px)"><span class="hero-word">amen</span></h1>
<div class="verse" style="margin-top:26px"><p>God, thank you for writing your law on our hearts. Where it exposes us, meet us with <span class="key">mercy</span>. Make us fair, honest and kind this week. Amen.</p></div>
<div class="say"><b>Say this</b><p>Next time in Mere Christianity: if this law is real, what does it tell us about the universe? Take your cup and go well.</p></div>
<button class="btn" data-first>Back to start</button>
</div></section>
</div>
<!--D1-END-->
<!--DEVOTIONS-END-->

<footer><button class="arrow" id="prev" aria-label="Previous">&larr;</button><span id="count" style="font-weight:700"></span><button class="arrow" id="next" aria-label="Next">&rarr;</button></footer>

<script>
const D=[...document.querySelectorAll('.devotion')],sel=document.getElementById('dsel');let S=[],i=0;
const dots=document.getElementById('dots'),count=document.getElementById('count');
D.forEach((d,n)=>{const o=document.createElement('option');o.value=n;o.textContent=d.dataset.title;sel.appendChild(o)});
function go(n){i=Math.max(0,Math.min(S.length-1,n));S.forEach((s,k)=>{s.classList.toggle('on',k===i);s.classList.toggle('past',k<i)});
[...dots.children].forEach((d,k)=>d.classList.toggle('on',k===i));count.textContent=(i+1)+' / '+S.length;S[i].scrollTop=0}
function load(n){document.querySelectorAll('.slide').forEach(s=>s.classList.remove('on','past'));S=[...D[n].querySelectorAll('.slide')];dots.innerHTML='';
S.forEach((_,k)=>{const b=document.createElement('button');b.setAttribute('aria-label','Slide '+(k+1));b.onclick=()=>go(k);dots.appendChild(b)});
sel.value=n;try{history.replaceState(null,'','#d'+(n+1))}catch(e){}go(0)}
sel.onchange=()=>{load(+sel.value);sel.blur()};
document.getElementById('next').onclick=()=>go(i+1);document.getElementById('prev').onclick=()=>go(i-1);
document.querySelectorAll('[data-next]').forEach(b=>b.onclick=()=>go(i+1));
document.querySelectorAll('[data-first]').forEach(b=>b.onclick=()=>go(0));
addEventListener('keydown',e=>{if(e.target===sel)return;if(e.key==='ArrowRight'||e.key==='PageDown')go(i+1);if(e.key==='ArrowLeft'||e.key==='PageUp')go(i-1)});
let tx=0;addEventListener('touchstart',e=>tx=e.touches[0].clientX,{passive:true});
addEventListener('touchend',e=>{const d=e.changedTouches[0].clientX-tx;if(Math.abs(d)>70)go(i+(d<0?1:-1))},{passive:true});
document.querySelectorAll('.card:not(.open)').forEach(c=>c.onclick=()=>c.classList.toggle('open'));
document.querySelectorAll('.flip').forEach(c=>c.onclick=()=>c.classList.toggle('open'));
document.querySelectorAll('.steps li').forEach(l=>l.onclick=()=>l.classList.toggle('done'));
document.querySelectorAll('.say,.ask').forEach(b=>b.addEventListener('mouseenter',()=>{b.style.setProperty('--ty','0px');const r=b.getBoundingClientRect(),c=(104+innerHeight-124)/2,off=c-(r.top+r.height/2);b.style.setProperty('--ty',Math.max(-60,Math.min(60,off*.25))+'px')}));
// bubbles
const cols=['#e0267f','#19a7d8','#2a2430','#e2a22b','#fff'],o=document.getElementById('orbs'),B=[],still=matchMedia('(prefers-reduced-motion:reduce)').matches;
function mk(init){const s=16+Math.random()*74,d=document.createElement('div');d.className='bub';d.style.width=d.style.height=s+'px';o.appendChild(d);
B.push({d,s,x:Math.random()*innerWidth,y:init?Math.random()*innerHeight:innerHeight+s+Math.random()*120,v:.25+Math.random()*.55,ph:Math.random()*6.28,sw:6+Math.random()*16,popY:Math.random()<.2?innerHeight*(.15+Math.random()*.55):-1,p:-1})}
for(let k=0;k<16;k++)mk(true);
function tick(){for(let k=B.length-1;k>=0;k--){const b=B[k];if(b.p<0){b.y-=b.v;b.ph+=.012;if(b.popY>0&&b.y<b.popY)b.p=0;else if(b.y<-b.s-10){b.d.remove();B.splice(k,1);mk(false);continue}}else{b.p+=.12;if(b.p>=1){b.d.remove();B.splice(k,1);mk(false);continue}}
const sc=b.p<0?1:1+b.p*.45;b.d.style.opacity=b.p<0?1:1-b.p;b.d.style.transform=`translate(${b.x+Math.sin(b.ph)*b.sw}px,${b.y}px) scale(${sc})`}requestAnimationFrame(tick)}
if(still){B.forEach(b=>b.d.style.transform=`translate(${b.x}px,${b.y}px)`)}else tick();


// click burst
addEventListener('click',e=>{for(let k=0;k<10;k++){const p=document.createElement('div');p.className='pop';const s=8+Math.random()*10;const a=Math.random()*6.28,r=40+Math.random()*60;p.style.cssText=`left:${e.clientX}px;top:${e.clientY}px;width:${s}px;height:${s}px;background:${cols[k%5]};--x:${Math.cos(a)*r}px;--y:${Math.sin(a)*r}px`;document.body.appendChild(p);setTimeout(()=>p.remove(),800)}});
// timers (one per devotion)
document.querySelectorAll('.timer').forEach(tm=>{let t=null;tm.onclick=()=>{if(t)return;let s=30;tm.textContent='Quiet… '+s;t=setInterval(()=>{s--;tm.textContent=s>0?'Quiet… '+s:'Amen. Tap next';if(s<=0){clearInterval(t);t=null}},1000)}});
// open the devotion in the link (#d2) or the newest one
const h=/^#d(\d+)$/.exec(location.hash);load(h&&D[h[1]-1]?h[1]-1:D.length-1);
</script>
</body>
</html>
```
<!--TEMPLATE-END-->
