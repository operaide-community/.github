<p align="center">
  <img src="https://raw.githubusercontent.com/operaide-community/.github/main/profile/operaide-icon.svg" alt="Operaide" width="120">
</p>

<p align="center"><strong>Operaide Community Edition</strong> · Free · Invite-only</p>

<h1 align="center">Your app factory. Under your rules. Self-hosted.</h1>

<p align="center">Everyone builds apps now: you, and your colleagues by vibe coding.<br>Each app gets deployment, login, roles and the other -ilities from the factory, as TypeScript you can review.</p>

<p align="center">
  <a href="https://community-edition.operaide.ai/apply"><strong>Request access</strong></a><br>
  <sub>Use your personal GitHub account</sub>
</p>

## Describe it, and it runs

<!-- App Builder screenshot: chat on the left, the running app in the preview on the right, the open model in the input field. -->

You or your colleagues describe the app in the App Builder. The agent plans, writes the code, checks it, and deploys it to a private dev instance after every message. The App Builder is an extension you switch on, and only the roles you choose can build and publish. What is built runs as a development app; production is a separate area.

## What you review

What the agent writes is TypeScript. Open it in Studio, read it, change it:

```typescript
import { aktorConst, createAktorComposition, createAktorFunction } from '@operaide/aktor';
import { aktorAICall, aktorAISettingProviderModel, aktorMessagesFromSystemAndUser } from '@operaide/ai';

const aktorCombine = createAktorFunction('aktorCombine', ({ summary, sentiment }) => ({ summary, sentiment }));

export const aktorAnalyze = createAktorComposition('aktorAnalyze', ({ text }) => {
    const summary = aktorAICall({
        messages: aktorMessagesFromSystemAndUser({ system: aktorConst('Summarize in one sentence.'), user: text }),
        providerModel: aktorAISettingProviderModel(),
    });
    const sentiment = aktorAICall({
        messages: aktorMessagesFromSystemAndUser({ system: aktorConst('Reply: positive, negative, or neutral.'), user: text }),
        providerModel: aktorAISettingProviderModel(),
    });
    return aktorCombine({ summary, sentiment });
});
```

`summary` and `sentiment` do not depend on each other, so both LLM calls run concurrently. No `Promise.all`. The platform draws the graph from the same code:

![The graph of a multi-agent app, drawn by the platform from its code](https://raw.githubusercontent.com/operaide-community/.github/main/profile/platform.png)

## Where it runs

One Docker container on your infrastructure.

Build with any model you trust with your code. Run on a model that keeps your data in the house: self-hosted on your own hardware, or hosted in Germany. Because the architecture is decided in advance, building does not need a frontier model either.

Development: yes. Production: no. The Community Edition is licensed for building, testing and demos, paid client work included. Running a business on it needs a paid license.

## How to get in

1. **Request access** with your email and GitHub handle.
2. **We invite you** to this organization: the repositories, the handbook, the Discord server, and €5 of credit for models hosted in Germany from our partner Noirdoc.
3. **One container, one description.** `docker compose up`, add a model, switch on the App Builder, describe your first app.

This organization goes public after the preview, with everything written in it. Write as if it were public today.

<p align="center">
  <a href="https://community-edition.operaide.ai/apply"><strong>Request access</strong></a><br>
  <sub>Use your personal GitHub account: managed enterprise accounts cannot be invited</sub>
</p>

<p align="center"><sub>Operaide is a brand of objective partner AG · <a href="https://operaide.ai/legals/imprint">Imprint</a> · <a href="https://operaide.ai/legals/privacy-policy">Privacy Policy</a> · <a href="https://operaide.ai/contact">Contact</a></sub></p>
