# Claude

Connect Teachfloor to Claude from the connector directory.

## Connecting

Teachfloor is listed in Claude's connector directory:

**[claude.ai/directory/teachfloor](https://claude.ai/directory/teachfloor)**

Open it and add the connector. Claude registers itself and opens a Teachfloor consent screen where you choose the organization to connect and approve the access. Nothing needs to be set up on the Teachfloor side first — no application to register, no key to paste.

You can also add it by hand: in Claude, add `https://mcp.teachfloor.com` as a custom connector. The result is the same, it is just more typing.

See [Getting Started](../getting-started) for what the consent screen grants and how access is scoped.

## Asking for Things

Once connected, the tools are available to the conversation. Claude picks the ones it needs:

> *"Which learners in the onboarding course haven't completed the first module?"*

> *"Add a module called Week 3 to the compliance course, with a video lesson and a quiz."*

> *"Enroll everyone in the Design team into the new accessibility course."*

The last one writes, so Claude will ask before it runs.

## Troubleshooting

**The connector won't authorize.** Check that the organization's plan includes MCP — without it every request is answered with `403 Forbidden`, including the first one after consent.

**Tools are missing or greyed out.** The token carries the permissions of the person who approved it. A lecturer or assistant sees only the courses and members they can see in Teachfloor.

**It asks for confirmation on something that only reads.** Worth reporting — a tool that only reads should never need approval. Contact support through your Teachfloor dashboard.
