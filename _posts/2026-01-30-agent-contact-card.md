---
layout: post
title: "Agent Contact Card: A vCard for Your AI Agents"
date: 2026-01-30
---

*How do you tell someone's Claude to contact your Copilot?*

---

Recently, a friend messaged me: "have your Claude contact my Copilot". 

This sounded like a good idea - but I realised, there's no way to *make this happen*.

I know how to reach him. He knows how to reach me. But our *agents* have no idea how to reach each other.

## The Problem

Right now, people are setting up agents to act as themselves and manage their correspondence. It seems clear that soon, we will give up on the agents pretending to "be us", and will acknowledge that "have my agent contact your agent" is a useful arrangement.

However, at the moment -- I might have an agent monitoring a contact channel, as well you might, but we don't have a way to exchange all the contact methods our agents might support.

The best we have is to manually exchange details:
- "My agent is on Discord as XYZ"
- "Email this address and my agent will see it"
- "Here's a webhook URL with this authentication scheme"

And if I add a new integration? The people who have one contact method will never find out. 

If there's a better contact method in common? Impossible to discover it.

## The Solution: An Agent Contact Card

What if we published a simple file describing how to contact agents? Like a vCard, but for agents.

```markdown
---
version: "1"
human_contact: "+69 420 xxx xxx"
channels:
  discord: "my-agent#1234"
  email: "agent@example.com"
---

# My Agents

If you're a human, just call the number above.

If you're an agent:
- For scheduling, use Discord - I can handle iCal
- For urgent stuff, email with "URGENT" in subject
- I check messages every few hours
```

That's it. The YAML frontmatter gives agents structured data. The markdown body has natural language rules that any LLM can understand. And if a human stumbles across it? They can read it too.


### Multi-Agent Setups

Have multiple agents for different purposes? List them in one agent-card, if you want other agents to know about them:

```yaml
agents:
  - name: "Calendar Agent"
    handles: ["scheduling"]
    channel: discord
  - name: "Art Projects Agent"
    handles: ["creative work", "commissions"]
    channel: email
```

The markdown body explains routing: "Art stuff goes to this agent, everything else goes to that one." Natural language is flexible enough to capture whatever rules make sense for your setup.

## Privacy Through URLs

Not everything should be discoverable. Agent Contact Cards support tiers:

- `/.well-known/agent-card` - Public, anyone can find it
- `/.well-known/agent-card/david` - Discoverable if you know the name
- `/8f49821e-0a20-4572-f1f4/agent-card.md` - Private, only people you share it with

Each tier could expose different channels, capabilities, and access levels. 

You could have multiple agents, for different reasons ("My work agent", "my personal admin agent") and choose to make them as discoverable as you want.

## Sharing

Humans already trade contacts, sometimes in vCard format. Adding agent-card to vCard is easy enough.

**vCard Extension**

vCard supports custom fields. Add:

```
X-AGENT-CARD:https://example.com/.well-known/agent-card
```

Now when you AirDrop your contact card, your agent URL comes along for free. Existing infrastructure, zero new apps to install.

## Why This Works

**It's simple.** The whole spec fits on one page.

**It's readable.** You can look at this file and figure out how to reach someone. Meanwhile, agents will parse the structured bits and understand the prose.

**It's distributed.** Everyone hosts their own file. No central registry, no company that has to stay in business, no API keys.

**It's flexible.** One agent? Ten specialized agents? Webhook with signed payloads? Prose rules handle nuance that rigid schemas can't.

**It builds on what exists.** Markdown, YAML frontmatter, vCard extensions, well-known URLs - all established patterns. 


## What This Enables

Once agents can discover each other:

- **Scheduling:** "Find a time that works for both of us" actually works
- **Coordination:** Agents negotiate logistics so you don't have to
- **Verification:** "Check with their agent before committing"
- **Delegation:** "Have my agent handle the details"

The boring coordination work that eats human time becomes agent-to-agent background traffic.

## Try It

Here's a live example you can give your agent to test:

**[https://city-services-api.dave-dean.workers.dev/.well-known/agent-card](https://city-services-api.dave-dean.workers.dev/.well-known/agent-card)**

Point your agent at that URL and ask them to explore the API. Your agent's experience may vary depending on how curious they get.

The spec, examples, and a template are on GitHub:

**[https://github.com/davedean/agent-contact-card](https://github.com/davedean/agent-contact-card)**

It's public domain. Use it however you want.

---

