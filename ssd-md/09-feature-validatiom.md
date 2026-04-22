It's nice to have a feature ready, but don't merge yet.
In this last feature step, we collaborate with the agent
to review the work.
Fortunately, programmers have good tools
for this validation task.
After all, programmers have long spent
several hours per week on code review.
Start with the commit view. Let's go through the changes.
Focus your review on high-level concerns
like whether the features work and reflect the spec,
rather than details like which CSS classes were implemented.
For example, the Home.tsx is minimal.
Nano really meant nano.
We want a layout component
with clear landmarks for header, main, and footer,
but do these as sub-components.
And create a CSS file.
As it turns out, this mistake in the code
flowed from a mistake in the plan.
We didn't ask for this.
Let's ask the agent to do the fix,
thus correcting both spec and implementation.
This process that consists of the agent generating something
and us verifying it is known as the human in the loop.
Because agents are so fast at writing code,
software developers have lately been talking about cognitive debt,
the mental load of tracking what your code is doing
and how it has evolved.
That is why in order for this process
to be fast and easy to control,
and to reduce cognitive debt, the changes should be manageable.
The agent is making our changes, but
also validating that these changes didn't break anything.
Good news, the agent finished and gave us a quick summary.
You can also see this in the editor.
The feature plan was changed to add a Group 5
to list these new steps.
The code now has a layout component.
and a CSS file was added.
It looks like the three subcomponents
were just put in the same file,
but that doesn't follow conventions.
Let's put these in their own files
using the IDE move tool.
These kinds of changes are easy in editors
and we've always been good at this.
No need for an agent, right? Let's just do it.
Not so fast. This can lead to drift
where other artifacts, specs, readmes, get out of sync.
Let's ask the agent to fix any mentions
and to make sure this doesn't waste some time later.
All changes are done. Did we break something?
Tests would help, but the agent didn't install our testing package.
Since this is needed for all features,
we'll cover it in the next lesson.
Now we're ready to mark this
work as complete and merge our results.
Notice that the part of the constitution
was updated alongside the feature in this branch.
How to handle versioning of the specs and how to associate
which specs created which code changes
is an evolving topic in the community.
The change to the roadmap is small,
just checking off a step in the roadmap.
So it's okay to keep these updates on the same branch.
If the overall constitution update is more complicated,
it might be better to do this in a separate branch.
Now we're ready to mark this work as complete
and merge our results.
We just use Spec-Driven Development
to develop and validate our first feature.
But wait, shouldn't we take another look
at the project's broader scope?