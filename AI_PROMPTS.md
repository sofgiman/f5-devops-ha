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
4. 
```
Let's fix task 4 (CI is red). This is the F5 DevOps challenge repo - see README.md
for the task, and app.py and the tests are off-limits.

Work it the way I like:
1. Diagnose the defect first. Once you find the symptom, DON'T edit RCA.md or
   AI_PROMPTS.md and don't just run everything yourself - show me the error and
   give me the commands so I can reproduce it on my own. Then wait till I confirm
   I understand it.
2. After I prompt you to continue, show me 2 options to fix it, each graded 1-10
   at a senior level, with your recommendation. Let's discuss before you touch
   anything.
3. Only once I say go, apply the fix and verify run-tests.sh / CI is green.

Git flow (I drive the merges): new branch off latest main named fix/task4-...,
conventional commit (no co-author/attribution trailers in the commit), push, open
a PR into main, and I'll tell you when to merge. Writing style: no em dashes, plain
"-". CLAUDE.md stays untracked.
```

5. I added a debugging routine, so for every task/bonus task to `claude.md` I would just say: "lets start the `minimize proxy image` bonus task using the debugging routine". This is the routine:
```
# Debugging routine (every task, including bonus)

- Diagnose first. When you find the symptom, DON'T edit `RCA.md` or `AI_PROMPTS.md`
  and don't just run it all yourself - show me the error and give me the commands
  so I can reproduce it on my own. Then wait until I confirm I understand it.
- After I prompt you to continue, show 2 fix options, each graded 1-10 at a senior
  level, with your recommendation. Discuss before touching anything.
- Only once I say go: apply the fix and verify (`run-tests.sh` / CI green).
- I drive git. You branch/commit/push/PR when I ask; I decide when to merge.

```

6. 
``` 
Ok, This task changed a stuff related to previous tasks.
Especially  task minimize proxy image bonus - we changed the image:

add a small  short lines (max 2-3) Update at the end of the section, which says we updated the image and the reason - also show the new image size with the same format.
Also, add in the @RCA.md  small explanation on what we did here to  stop running as root. 
Explain in short 1 liners:

1. the unprivileged nginx image
2. @proxy/nginx.conf  change,
3. compose change
```

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
