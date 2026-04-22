We successfully implemented the first feature.
Can't wait to do it again and again.
Don't rush into it. Take a step back
and reflect with some replanning.
You have to run slow to run fast.
We have a workflow step called validation.
Tests are good for validation, but we
didn't give the agent all our testing preferences.
Let's make a replanning branch
to update the tech stack in our constitution.
As you saw in the previous video,
spec versioning is an evolving topic.
The constitution is a living document.
It's good practice to make updates
to it in its own separate branch,
so you can keep track of
which versions of it produced which code.
We'll use a prompt to state our policy.
It looks like the agent updated the package.json
just to have a script, no dependency.
Something to note for later, when we do run tests.
This change to add a testing framework
applies to future code.
But let's tell the agent to update existing
feature specs and implementation
based on this constitution change.
After these two prompts, we have some good changes.
The agent set up this testing change,
but didn't write the tests themselves.
Let's tell the agent to write some new tests.
With this setup, we can run tests conveniently in our editor.
Let's run under the debugger to give us a chance
to step through code execution as
part of the human in the loop.
Looks good. We now have tests as part of validation.
Let's commit.
Sometimes we realize we want to
do something differently in the product plan.
For example, after implementing the first feature,
we got an update from the product manager.
Because 40% of our users are on mobile,
we want to emphasize a responsive design.
Let's go to the agent.
Tell the agent we want responsive design
and to correct the product specs and
feature specs as well as any code.
Since we're so early on in development,
this update is a small change, so it makes sense
to directly implement it during replanning.
But if the new work is big, it's better to schedule it
on the roadmap as its own feature phase,
instead of just doing it in replanning. Use your judgment.
Remember, we want the specs to capture decisions, not just the code.
We want to try to keep
both in sync to help team communication.
The agent has implemented our responsive fix
with updates not just in code,
but also to the product and feature specs.
Time for us to do our part and review the changes.
Let's go through the diffs. This looks good. Let's do a commit.
Remember, working in small steps
with frequent commits keeps the review
from overloading your brain.
While this first part of replanning is working,
let's step back and do some project housekeeping.
For example, let's revisit the project roadmap
and look at the next task.
Does it still make sense?
The roadmap shows the next feature
which we cover in the next lesson.
Is it still the correct one?
Looking through the roadmap, it looks like features two, three,
four, and five kind of hang together.
Let's update the roadmap to tackle those in one step.
We'll commit and start on this new feature shortly.
The replanning step can be about a feature or the whole project.
But replanning can also be about improving your spec-driven development workflow,
across projects, across your organization.
For example, maybe you have a few non-technical stakeholders
that want to monitor the project's progress.
You'd like it to update a changelog
on each merge to main.
Most AI coding agents support skills,
a package of instructions and resources
providing the agent new capabilities and expertise.
Skills are great for definable, repeatable workflows
that require context specific to your project or organization.
A skill would be great for implementing a changelog update.
You could write this skill by hand,
but many agents actually have skills
that help you write a skill.
Let's use the agent to help us write
and maintain a changelog skill.
The agent gets to work.
One thing for us to consider is
this skill unique to this project?
Or should it be a standard part of all projects?
This is a style choice and you'll
gain better understanding as you use it.
You'll learn to automate your SDD workflow with skills.
For example, your validation step might include updating the readme,
linting, formatting, test writing,
and other quality checks.
You can work with your agent to package those
into a validation