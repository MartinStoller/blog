| Date       | Target Audience    |
|------------|--------------------|
| 01.02.2027 | Myself, Data Engineers / Architects |

# Inheriting a Databricks Platform: Lessons from a First-Time Data Architect

I recently started an exciting new role as Data Architect and Tech Lead at a large engine manufacturer. My main goal: establish and manage Databricks as the company's central, enterprise-wide data platform.

I've never held an architect role or title before, so I'm excited, but also a little nervous, about this new challenge. In this post, I want to share my experiences so far and reflect on the challenges I've faced and the lessons I've learned. I hope you find it useful, or at least interesting. If nothing else, it serves as an exercise in self-reflection for me.

### The Setup and Historical Context

The idea behind the project is to build a company-wide data platform that serves a wide range of teams and use cases. The project kicked off about a year ago, with Databricks on Azure chosen as the core technology. During that first year, the team consisted of a team lead/project manager and five nearshore data engineers. Progress fell short of expectations, and after a year the following problems and goals had emerged:

- **Costs:** The platform is more expensive than expected, and it's not yet clear why. Reducing infrastructure costs will be a major goal.
- **Governance:** There is no clear governance or data access model in place. (Details unclear)
- **Usability:** Several teams report that they can't work with the platform effectively, whether that's getting their data in and out, using compute resources, or something else.
- **Modernization:** A rather vague goal to "make use of the new AI and agentic features Databricks has released in recent years, apply them where it makes sense, and use Databricks in a modern way."

It became clear to the company that the team was missing a tech lead or architect to take technical ownership of the project. That's where I came in: they brought me on as an external consultant until they find a long-term internal solution.

### Onboarding

After five years in IT consulting, I've had my fair share of onboardings, and by now I have a good sense of what makes a good one and what to focus on first:

- **Get to know the team in one-on-ones.** Understand each person's role, background, and area of expertise, and get off to a good start with everyone. Use these conversations to learn about the project's history and its *actual* problems. The list above is what I was told in the interview, and that picture often differs considerably from reality. Most importantly: write everything down!
- **Understand the architecture of both the system and the organization.** Draw diagrams to make sure you've really understood it. As a bonus, you end up with useful documentation.
- **Become productive as quickly as possible.** Start working on real issues and tickets from day one. Pick easy tasks first and ask for help when you get stuck. Doing real work is by far the fastest way to get up to speed, and there will never be a better time to ask lots of "stupid" questions.

###### Get to know the team in one-on-ones
I prepared the following key questions for each conversation while remaining open to letting each conversation flow in whichever direction:
- Who are you and how do you see your role in this project?
- What are your biggest strengths in regards to this project?
- What are currently the biggest/most pressing issues in your eyes?
- What would be your expectation for my work in all of this?

Summarizing all conversations, what i discovered was this:


