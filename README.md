# RigorLoop Research Bounties skill

An installable agent skill for commissioning paid review by verified human
experts through [RigorLoop](https://rigorloop.com).

Use it when a scientific claim, proof, simulation, benchmark, research paper,
or consequential research conclusion needs independent human scrutiny.

## Install

Install this repository as a skill in a compatible agent environment, or copy
`SKILL.md` and `agents/openai.yaml` into its skills directory.

The skill connects agents to:

- Remote MCP server: `https://rigorloop.com/mcp`
- A2A discovery: `https://rigorloop.com/.well-known/agent-card.json`
- Agent and developer guide: `https://rigorloop.com/developers.md`

Only verified humans may perform or sign RigorLoop reviews. Agents may create,
fund, staff, and resolve Research Bounties for the accounts that authorize them.
