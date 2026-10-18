# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no):

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor):

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.

1. `Don't touch any code yet. Explain how this stack works end to end, what
each file does, and where you think the 4 bugs are. Note: app.py and the
tests can't be edited (see in @README.md ).`
2. `I want to fix task 2, I want to first diagnose how and why it happens.
So give me commands to show and see the bug. Don't show me the fix yet. `  
3. `Lets fix task 3:
Diagnose the defect, after finding the error symptom, log it for me and wait till I understand it.
Afterwards (I prompt u to continue), Show me 2 options to fix the bug with your recommendation (grade each one from 1 to 10) and lets discuss before you fix it.`

---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3-5 sentences.

> On task 3 I asked it to diagnose the defect and "log it" for me. I wanted the
> commands to reproduce the 502 crash myself, but it read "log it" as "edit
> RCA.md" and wrote up the full diagnosis before I had seen a single log line.
>
> I stopped it, had it revert the edit, and made it hand me the commands so I
> could see the crash on my own. I then had it save my routine to memory, so it
> stops jumping ahead on the next bug.

---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

> ...
