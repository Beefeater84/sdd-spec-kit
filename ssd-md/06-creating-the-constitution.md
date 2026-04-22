You need to tell your agent about your project,
the mission, audience, and other decisions.
In this lesson, we'll work together
with the agent to write the Constitution.
What really is your project?
Say you're working on a new web app for your company.
What's the core idea behind its development?
How does it fit within your company's preferred tech stack?
What features are planned?
These three foundational principles form the Constitution,
a global set of high-level requirements
that will guide future feature development
and explain the project's shape to stakeholders.
For example, you have so many choices for your tech stack.
You'll want to narrow down your options based on appropriate tradeoffs
and what you use at your company.
We need to write this down.
for the agent, for your teammates, for the future.
But we don't write it alone.
We write it in a conversation with the agent.
You'll be surprised at the great questions it will ask.
architecture patterns you hadn't considered,
external packages that already do the work,
or tradeoffs. For example,
speed versus data fidelity.
The mission, tech stack, and roadmap aren't just properties
of a Greenfield project you're starting from scratch.
Existing codebases use these three pillars too.
We will talk about bringing the SDD workflow to existing projects
in just a few videos from now.
Now, let's talk about completely new projects.
We are writing AgentClinic, a place for AI agents
to get relief from their humans.
AgentClinic is a fun parody of the popular learning project Pet Clinic.
Coding agents are doing a lot of work.
It's stressful. Hallucinations,
context rot, memory issues and
co-worker sub-agent coordination
all take a toll on agent health.
Let's build a clinic where they can get help with their issues.
It's a full stack web app with a Next.js back end
and a React front end.
The app allows you to manage appointments,
ailments like hallucination,
and treatments like a context infusion.
Let's get started on the development.
Let's take a look at how you might go about writing
a detailed spec for a project like this.
We're about to look at a lot of text,
but this technique will pay off for you down the line.
Yes, it's common to have a spec this long.
This document contains a lot of information
that the average Claude code instance
isn't going to know.
For example, lines 37 through 43
real problems we're trying to solve.
Agents can check themselves in via API
and their issues persist over time.
So we can see how effective treatments are.
Line 57. The visit lifecycle is mapped out
including a TRIAGE step to route them to the right place
and a FOLLOW-UP with three possible states.
Line 379, we also have an ailment catalog
plus the ability to have custom ailments.
Ailments have codes and severity levels,
which can be modified if the agent, for example,
handles medical or legal data.
Line 430. We'll auto-create a custom ailment
when symptoms don't match any existing ailment
above a 0.6 similarity threshold.
Line 581, we specify the exact algorithm used
to update a treatment's effectiveness,
Let's chat with the agent to see
if there's anything that could be cleared up.
We've already done a few rounds of this,
but there will still be some lingering questions
and you could pretty much always clarify more.
The agent says there's a threshold inconsistency in the diagnosis flow.
Let's scroll to line 143 and check it out.
It looks like three is actually the correct option.
If a diagnosis is between 0.4 and 0.6 confidence,
then it's included but flagged as uncertain.
So, this is a great sign. There's no issue with the spec.
We're just confirming our understanding with the agent.
For this question, let's just say having an unprotected dashboard is fine
because we expect to deploy this privately
in a secure environment to our company.
For the LLM, let's leave that configurable
because we don't know how soon a new model will be released.
For archiving, we're not sure about this behavior right now,
so let's choose the first option, soft-delete,
which gives us the most flexibility later.
We'll be moving forward with a somewhat more pared down mission document.
In the repo, you'll find both.
You can choose to use either version.
For a challenge, try the most complex one shown here,
to follow the course exactly, use the shorter ones.
Next, let's take a look at the tech-stack file.
This document separates out the key architecture decisions from the mission document.
The product can be implemented in many
ways, but some are better than others.
Let's walk through some key ideas to include.
Line 51, the full pipeline that happens
when an agent hits the /visits route with a post request
and involves validating the request, searching for previous visits, and so on.
Line 588. The database schema for all the tables.
This is great to have in advance because
it's such a headache to update the schema later. Line 738,
A list of all possible treatments like Temperature Reduction.
Then on line 706, we see which ailments get which treatments.
Line 899. We go through some basic smoke tests like
checking to make sure that chronic ailments are detected.
Again, let's see if anything needs to be cleared up.
This is a great question