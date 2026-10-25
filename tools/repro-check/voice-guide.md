# Voice guide: how I talk upstream

<!--
THIS IS THE PART YOU WRITE (new this week). Live mode reads this file
before any comment of yours goes out the door; eval mode ignores it
entirely, because your voice is yours and carries no gold labels.

This is not etiquette. "Be polite and concise" is advice for everyone
and therefore rules for no one. Write rules YOU need, in your own
words, each one concrete enough that the skill can hold a draft
against it and say which rule it breaks.

Three sections. Fill all three.
-->

## Who I am in threads

I'm Mohannad, a 2026 CS graduate (UNL) taking CodePath AI301.
Path Review is where I make my first open-source contributions, so I'm new to this codebase.
Expect exact commands, pasted output, and plain status; when I don't know something yet, I say so.

## Rules I write by

### Rule: Promise work, not results

Until the work is done, I promise the next step and the report, never a fix, an outcome, or a date.

- Wrong: "I'll have a fix for the orphaned embeddings up by the weekend."
- Right: "Next I'm reproducing this through the `DELETE` route and will post the output here."

### Rule: Show, don't characterize

When I say something happened, the output that shows it is in the same comment.

- Wrong: "Deleting a profile is basically broken, the embeddings just stay there."
- Right: "`DELETE /profiles/{id}` returned 204 and the follow-up GET returned 404, but the `profile_{id}` collection still held 1 chunk and `HybridRetriever.retrieve()` still returned it."

### Rule: Guesses wear a label

A cause I have not proven in code or output is marked as a guess, and I say what I have not checked yet.

- Wrong: "The bug is that Chroma's delete silently fails."
- Right: "I think nothing on the delete path calls the vector store at all: `delete_profile` in `core/services/profile_service.py` only deletes DB rows. I haven't traced every caller yet."

### Rule: Credit, then add

When others already posted on the issue, I name their work and say what mine adds; I never just agree.

- Wrong: "Same as above, can confirm on my machine."
- Right: "PRAYAG-PATEL-1998 and Lizyum showed the chunks survive `delete_profile()`. This run goes through the HTTP `DELETE` route and also checks that the retriever still returns the chunk."

## Things I never post

- Dates or ETAs ("PR by Friday", "fix coming shortly").
- "+1", "same here", or "can confirm" without my own output.
- Emoji, exclamation marks, or superlatives I can't back with a number.
- A judgment that a classmate's repro or PR is wrong. If my evidence differs, it goes in my own comment as evidence.
