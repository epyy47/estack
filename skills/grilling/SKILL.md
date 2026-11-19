---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user says "grill me", "/grilling", or wants to stress-test their thinking.
---

Interview me relentlessly about every aspect of this until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer. Put the recommended option first in the `AskUserQuestion` options and end its label with `(Recommended)`.

Ask the questions one at a time using the `AskUserQuestion` tool, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a _fact_ can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The _decisions_, though, are mine — put each one to me and wait for my answer.

Do not act on it until I confirm we have reached a shared understanding. When I confirm, print a numbered decision log: one line per decision with the answer chosen. That log is the input for the plan or the next skill.
