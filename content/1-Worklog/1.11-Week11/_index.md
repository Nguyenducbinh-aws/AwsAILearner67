---
title: "Week 11 Worklog"
date: 2026-11-16
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---


### Week 11 Objectives:

* Deploy the Cost Guard workshop and test it.
* Update the workshop, run the demo and write the report.
* Read about bringing AWS Transform into GitHub Copilot through MCP.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | - Deploy the Cost Guard workshop | 11/16/2026 | 11/16/2026 | |
| 3 | - Run the tests for the workshop  | 11/17/2026 | 11/17/2026 | |
| 4 | - Update the workshop after testing <br>- Run the demo | 11/18/2026 | 11/18/2026 | |
| 5 | - Write the report <br>- Clean up resources after the demo is done | 11/19/2026 | 11/19/2026 | |
| 6 | - Read the blog about bringing AWS Transform into GitHub Copilot with MCP | 11/20/2026 | 11/20/2026 | <https://aws.amazon.com/vi/blogs/dotnet/bring-aws-transform-into-github-copilot-with-mcp/> |


### Week 11 Achievements:

* Deployed the Cost Guard workshop and ran the tests on it.

* Updated the workshop after testing and ran the demo.

* Wrote the report.

* Cleaned up the resources after the demo was done, so the lab does not keep costing money.

* Learned from the blog how to bring AWS Transform into GitHub Copilot through MCP:

  * AWS Transform is an agentic AI service that analyzes and transforms whole applications, infrastructure and code, and the AWS Transform MCP server lets you call it in natural language from Copilot agent mode without switching to the console
  * The blog's example transforms a .NET Azure Functions app (HTTP, Queue and Blob triggers) into AWS Lambda with the Lambda Annotations Framework and generates a SAM template
  * AWS Transform pauses for approval before destructive actions such as push, merge and delete, and flags source constructs that have no direct AWS equivalent instead of generating broken code
  * A finished transformation can be saved as an Agent Skill (a SKILL.md file) and reused in tools such as GitHub Copilot, Kiro and Claude Code
  * Use least privilege for the IAM role that the transformation uses, and run sam delete to remove the resources at the end
