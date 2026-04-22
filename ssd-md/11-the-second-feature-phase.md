We have an improved constitution
and an improved workflow. We're getting better.
But before moving on to the next roadmap feature,
let's tackle some strategies to deal with a common pain point.
AI fatigue.
Agents can generate a lot of code with a lot of changes.
This massive amount of code makes
the human in the loop validation exhausting.
So much to review.
To fight this confusion, make sure you have
a clean division between each feature phase.
We should start each feature
in the right flow state.
Do I have unfinished work?
Did I merge the last feature branch to main?
Is the next roadmap item the right thing to do?
Did I clear the agent's context
to ensure the specs capture the intent
instead of memory snapshots?
And to let it focus its
limited context budget on the next work.
Take a moment to make a good start to help yourself finish.
We've done all this. Let's clear the context
and start the next feature, agents and ailments.
As a note, we keep typing the same thing.
We'll talk about how you can streamline this repeated prompting
into a custom workflow in a future lesson.
As before, the agent asks good questions.
The first one is about whether to
keep all these features in one phase.
We already decided to do this together in the previous lesson,
so we'll say yes. For the migrations, we'll choose plain SQL.
For validation, let's select one, two, and three.
That's good. Now, let's submit.
When the agent drafts the spec,
we can see some of its choices.
The agent's reasoning process is useful to watch.
If you're using an agent with verbose mode,
that could give you even more insight
into its intermediate ideas.
Looks good, but we want to make
one small change to capture our intent.
We want to use PicoCSS
as our CSS framework.
Next, we review the three files in the feature spec,
starting with the requirements.
Let's ask the agent to change the requirements
to make sure any other files reflect the change.
When the work is finished, let's commit the feature spec.
This time, we'll let the agent come up with the message.
With the spec for the second feature ready, let's go to implementation.
The division between the planning phase and the feature phase
helps us to not overflow context,
hours and agents.
If the feature seems too big to implement all at once,
ask the agent to implement part of the feature plan first.
This will keep the size of the changes manageable.
Our agent has implemented the phase two feature.
We watched the agent's progress,
so we have a rough idea of what was done.
But that's not enough.
Slow down, do some thinking.
Let's do an in-depth review of the changes.
Everybody has their own style on
how much to review the agent's results.
To play to the agent's strengths and reduce AI fatigue,
stick to higher level requirements.
Avoid nitpicking stuff like variable names.
Just make sure it creates code
that you can commit under your name.
For example, it used inline type information for the props.
We'd like the prop definition in a standalone TypeScript type.
Let's tell the agent to fix this everywhere in the code.
You probably didn't include this decision in your spec.
Remember, an omission such as extracted prop types isn't a failure.
You are evolving the spec as you discover new details
and capturing that leads to better future results.
We're finished with our review.
This is a good moment for another small step commit.
Last step validation.
Let's review what the agent plans to do
for the validation step in this feature spec.
Looks good. Let's do our validation.
Remember, we want to make sure the
changes are good by running the application.
But we also want to prevent cognitive debt
by validating that we understand these changes.
Tests are a great place to work through the flow.
Let's read some tests and run them
under the debugger as exploration.
Sometimes you need to validate that you weren't lied to.
Tell the agent to spawn several sub-agents
to do a deep review of the entire project
with this feature change.
This deep review gives the agent more space
to think about the changes.
And using sub-agents preserves
the main agent's context window, rather than polluting it.
These sub-agents will come back with a number of issues
and recommendations. All these look good, so let's proceed.
The agent goes off, does a bunch of fixes,
runs tests, and gets our project into a good state.
Keep this in your tool chest.
The agent can usually find important issues during a second look.
Once we are finished with our validation,
our feature is complete. It's time to commit our feature.
As our last step on this feature,
let's use the change log skill
that we made in the previous lesson.
We've now documented the changes on this feature branch.
One more commit, then merge the branch.
In a previous lesson, we saw the replan step in between features.
Let's do a quick version of this by looking at the roadmap.
Does the next feature still seem right?
Looks good. Our project is in
good shape, ready for the next feature.
This clean break between features helps manage AI fatigue.
Now you can take a break for some coffee.
We're starting to see a repeatable SDD process for feature development.
One that captures our decisions.
In the next step, let's test that hypothesis.
Do we have enough to build an MVP?