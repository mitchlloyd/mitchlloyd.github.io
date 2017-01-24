---
layout: post
title:  "Be More Productive by Limiting Work in Progress"
date:   2017-01-24
mermaid: true
---

As a new associate at a Big 4 Accounting firm, I was shuffled between projects
and subjected to whatever ad-hoc project management strategy each team used.
Although it was often frustrating, it was also instructive to observe different
approaches to audit work. Differences in productivity and effectiveness between
teams were stark.

Productivity is the foundation of an audit business. It is the only way to get
ahead that doesn't involve playing zero-sum games with your clients, co-workers,
and the firm. Teams that fall behind are faced with the decision to "limit their
careers" within their firm or [take their chances with random PCAOB
audits](https://pcaobus.org/News/Releases/Pages/enforcement-Deloitte-Brazil-12-5-16.aspx).

When I started building software for a living, I found that productivity and
effectiveness varied even more than in the audit world. Forget "The 10x
Programmer"---I saw variances that couldn't be captured with positive integers.
Some programmers could accomplish things that others never could and, in some
cases cases, removing a programmer from a team could improve output.

I'm fascinated by the concept of productivity. Reading _Getting Things Done_
marked a life-changing shift in my career and learning about Kanban was another
leap. There are many facets of productivity but this article will,
appropriately, focus on one thing: minimizing work in progress.

The Trap
--------

The idea of finishing one valuable thing before moving on to another may seem
too simple to warrant discussion. However, I find the concept to be deeply
technical, challenging, and counter-intuitive.

This is the most common approach to executing audit work:

<div class="mermaid">
graph LR
  step1[Decide what work to do]
  step2[Gather evidence]
  step3[Perform testing]
  step4[Review testing]

  step1-->step2
  step2-->step3
  step3-->step4
</div>

This is a valid plan that highlights the dependencies between steps. After
all, you must gather evidence before you perform testing. The trouble begins
when teams naively attempt to finish step 1 for the entire audit before
proceeding to step 2.

The same trap exists in software projects. The most obvious way to think about
work is in a series of procedural steps, each one depending on the step before.

<div class="mermaid">
graph LR
  step1[Gather requirements]
  step2[Write specifications]
  step3[Implement specifications]
  step4[Test system]

  step1-->step2
  step2-->step3
  step3-->step4
</div>

By now most development shops recognize "waterfall" flows like these and know of
an alternative approach called "agile". And _of course_ they do "agile". But
still the idea of completing each step across a project before proceeding to the
next creeps into most environments.

Developers might insist that product owners document every edge case before they
begin working. Product owners may expect the final product to look exactly like
the Photoshop mock. Product owners, developers, and designers may all negotiate
on deadlines for completing slices of work (e.g. Design sign-off, Feature
Freeze). Often stake-holders don't want to see a feature until it's "completely
done".

On a smaller scale, the same mental trap grabs hold during software
construction. Programmers often tackle large features by breaking them
down into steps based on dependencies, building with a bottom-up approach.

<div class="mermaid">
graph LR
  step1[Build data model]
  step2[Build the API]
  step3[Build the UI]
  step4[Test feature]

  step1-->step2
  step2-->step3
  step3-->step4
</div>

The Tree
--------

More successful approaches begin with the end in mind and derive increasingly
granular milestones.

For our audit deadline on February 1st we want an opinion letter saying:

> In our opinion the financial statements
present fairly, in all material respects, the financial position of Some Company
as of December 31, 2016.

How can we get there?

<div class="mermaid">
graph TD
  n1[Opinion]
  n2[General Controls Effectiveness]
  n3[Balance Sheet Accuracy]
  n4[Profit & Loss Accuracy]

  n1-->n2
  n1-->n3
  n1-->n4
</div>

This planning process proceeds top-down until we've reached the end of a team's
expertise. Eventually we reach the 60-hour-a-week interns and associates that
type and click our tree of audit stories into existence.

Let's look at a more generic tree of milestones:

<div class="mermaid">
graph TD
  a-->b
  a-->c
  b-->d
  b-->e
  c-->f
  c-->g
</div>

When visualizing our work this way we can start to see the fallacies of the
batching approach. If we start working in a breath-first manner (first gather
all the evidence) we would progress like this:

<div class="mermaid">
graph TD
  a-->b
  a-->c
  b-->d
  b-->e
  c-->f
  c-->g

class e,d,f visited
classDef visited fill:white,stroke:#333,stroke-width:2px;
</div>

As we look at this snapshot of progress:

* We have nothing valuable done. If we stop the audit today we have no comfort
  for our opinion---the same as if we had never started.
* We don't know if there is any rework ahead. Perhaps we've made a mistake
  that impacts each of these tasks, but we won't know until we get higher in the
  tree.
* We've only performed one type of work so we can't predict how long it takes to
  complete milestones.
* We have 3 in-progress items to keep track of.

Instead we should prefer a depth-first approach.

<div class="mermaid">
graph TD
  a-->b
  a-->c
  b-->e
  b-->d
  c-->f
  c-->g

class b,e,d visited
class e,d finished

classDef visited fill:white,stroke:#333,stroke-width:2px
classDef finished fill:#eee
</div>

Now when we look at our progress:

* We have one substantial part of our audit done. If we have enough top level
  items done at the deadline, we might be able to finish the audit with
  confidence.
* The knowledge we gained from finishing this branch informs the work we do on
  other branches. We might do less substantive audit work because can rely on
  General Controls.
* We've learned a lot by exercising one loop of our procedural machine. Any
  feedback that we got during review can be incorporated into future work. We're
  in a better place to estimate how long things take.
* Nodes `e` and `d` represent work that we no longer need to think about, track,
  and organize.

The same risks and trade-offs apply in our software project. If we've designed
the data-model for all of our features but have nothing that users can interact
with then we're in a bad spot. We carry risk that our data-model is wrong, we
miss out on feedback on our process, and we incur the cost of maintaining
in-progress work.

Looking at our tree of milestones we can say that:

1. The higher a node is in the tree, the more valuable it is.
2. Each in-progress node we can reach from the root has a high cost.

This is Hard
------------

The first principle from the Agile Manifesto says:

> Our highest priority is to satisfy the customer through early and continuous
> delivery of valuable software.

Lean manufacturing teaches us to be sensitive to different types of waste in our
systems, many of which arise from work-in-progress.

Perhaps these ideas seems obvious in retrospect but they are clearly not the
way people _naturally_ approach work.

I frequently catch myself slipping into bottom-up workflows. Maybe this has
something to do with human optimism and hubris; many of us don't count on
mistakes and we think we know more than we do.  Recognizing that effective work
patterns _are_ hard to execute and can be non-intuitive is important if we care
about productivity. So I challenge myself and others to ask "how can we minimize
work in progress"?
