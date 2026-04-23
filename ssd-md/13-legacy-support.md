People say spec-driven development and even AI
are only good for greenfield projects.
But SDD is also good for existing legacy projects.
In this lesson, we'll introduce SDD to an existing project.
To make it easy, we'll start with a project you already know.
The AgentClinic MVP.
We'll make it a legacy by
starting on main without the specs folder
in a new Claude Code session
in a different project directory.
We have some project background in the README.md file
with some open work listed in TODO.md
Your legacy project might have full product plans
in issue trackers, spreadsheets, Word documents, etc.
Remember our workflow? We'll start with the constitution step.
We'll use almost the same prompt from lesson four.
This time, we'll tell it to
look for roadmap items in existing artifacts.
In this case, a to-do file.
Remember, the agent will discover and in a sense reverse engineer
the SDD artifacts from the existing code base.
The constitution will help align future code changes made by the agent
with what past devs have already created.
You can always add more context if you have it.
For example, you might have been dropped into this project
in order to improve its efficiency
while implementing highly requested features.
The process is the same, but the conversation might be richer
as the agent has more artifacts,
code, commits, documents, etc.
You'll see a lot of tool calls here
as the agent explores the code base.
Let's look at the constitution for our legacy project.
In the mission, we have an audience, the project idea,
and some extra information about the project.
The tech stack shows that Claude extracted
the project file structure, framework versions,
and the clarifications we were asked.
In the roadmap, we can see the work is organized in phases,
matching the to-do file. Let's create a commit
for the constitution files that we produced
with a commit message.
Remember that in SDD,
specs are part of your versioning strategy.
Our legacy project is now placed
on an SDD foundation.
From now on, the workflow is exactly the same
as we discussed in previous lessons.
First, we grab the next feature on the roadmap
and plan it in a conversation with the agent.
We'll be doing phase one feedback form.
Very good. We now have a branch and specs for this feature.
Let's review the feature spec.
The plan,
the requirements, and the validation.
Then we commit the feature spec.
After we have made any corrections by chatting with the agent,
we proceed to implement the feature.
Once the agent is finished, we do our validation.
To make sure the code is
good and the feature works as expected.
Our feature loop is done, we can commit.
And then merge the branch to main.
Now it's time for replanning. Make sure you give time for this.
Since you just added SDD into your project,
you may find a lot of things to tune.
We now have a legacy project
rebased onto Spec-Driven Development
without a bunch of manual work.
This gives us a well-documented flow, working in steps,
using engineering and staying in control.
The spec is now the memory
of the project, so it doesn't fade.