<p align="center">
  <img src="operaide-icon.svg" alt="Operaide" width="120">
</p>

<p align="center"><strong>Operaide Community Edition</strong> · Free · Invite-only</p>

<h1 align="center">Your AI app factory. Under your rules. Self-hosted.</h1>

<p align="center">Everyone builds apps now: you, and your colleagues by vibe coding.<br>Each app gets deployment, login, roles and the other <a href="#terms" title="What every app needs besides its purpose: deployment, login, roles, audit, tracing">-ilities</a> from the factory, as TypeScript you can review.</p>

<p align="center">
  <a href="https://community-edition.operaide.ai/apply"><strong>Request access</strong></a><br>
  <sub>Use your personal GitHub account</sub>
</p>

## Run is the platform. Build is an extension.

<!-- App Builder screenshot: chat on the left, the running app in the preview on the right, the open model in the input field. -->

Every instance runs apps, each with the same [-ilities](#terms "What every app needs besides its purpose: deployment, login, roles, audit, tracing"). Switch on Build, and the same instance makes them too:

- **Describe it** in the App Builder. The agent plans, writes the code, checks it, and deploys it after every message. The way in for colleagues who do not code.
- **Code it** in Studio, a VS Code-style IDE in your browser: by hand, or with Claude Code, Codex or Operaide Code, all built in.

The best practices are encoded in every workspace: the handbook, the rules for agents, types and lint checks. Whichever agent writes the code, it builds the app the same way.

One instance can do both. Or run production without Build, and build on a second instance: every app is an npm package, every instance has a built-in npm registry, and apps move between instances like any other package.

## The vibe coding trap

It always starts like this:

```typescript
const [category, summary] = await Promise.all([classify(mail), summarize(mail)]);
const order = await lookup(findOrderNumber(mail));
```

Then it needs a REST endpoint, an OpenAPI spec, login and roles, a safe place for the shop's API key, a trace of every step, and a deployment. Each one gets vibe coded a little differently, in every app.

In Operaide you write the same steps as a composition:

```typescript
const aktorTriage = createAktorComposition('aktorTriage', ({ mail }: { mail: Aktor<string> }) => {
    const category = aktorClassify({ mail });
    const summary = aktorSummarize({ mail });
    const orderNumber = aktorFindOrderNumber({ mail });
    const orderSummary = aktorLookupOrder({ orderNumber });
    return aktorTriageResult({ category, summary, orderSummary });
});
```

The rest is already there: the endpoint and its OpenAPI spec come from the schemas, login and roles from the platform, the key sits in a connector, and every step is traced and drawn.

![A run of the composition, step by step: the order lookup starts last and finishes first](triage-trace.gif)

<sub>The order lookup starts last and finishes first: 574 ms, while each LLM call takes over a second.</sub>

A composition is declarative: it describes the graph and does not run it. That is why there is no `await`, and why the platform can run independent steps in parallel.

The steps are [Aktors](#terms "One step of a workflow: a function, or a composition of other Aktors"), and the endpoint built from them is a [Reaktor](#terms "A REST endpoint of an app, built from Aktors, with an OpenAPI spec from its Zod schemas").

<details>
<summary><strong>The full example</strong>: connector, functions, compositions, endpoint</summary>

`shop-connection.ts`

```typescript
import { z } from 'zod';
import { registerConnectionType } from '@operaide/aktor';

// A connector: an admin enters URL and key once, in the platform
export const shop = registerConnectionType({
    type: 'shop',
    label: 'Shop',
    defaultConnectionName: 'shop',
    configSchema: z.object({
        baseUrl: z.string().url().describe('[label:Base URL]Where the shop API lives'),
        apiKey: z.string().describe('[label:API Key][secretFor:baseUrl]Sent to the shop with every request'),
    }),
});
```

`TriageMail.reaktor.ts`

```typescript
import axios from 'axios';
import { z } from 'zod';
import { aktorConst, createAktorComposition, createAktorFunction, registerReaktorDefinition } from '@operaide/aktor';
import type { Aktor } from '@operaide/aktor';
import { aktorAICall, aktorAISettingProviderModel, aktorMessagesFromSystemAndUser } from '@operaide/ai';
import { shop } from './shop-connection';

// Functions: plain TypeScript, sync or async, any npm package
const aktorFindOrderNumber = createAktorFunction(
    'aktorFindOrderNumber',
    ({ mail }: { mail: string }) => mail.match(/\bORD-\d{6}\b/)?.[0] ?? null
);

const aktorLookupOrder = createAktorFunction(
    'aktorLookupOrder',
    async ({ orderNumber }: { orderNumber: string | null }) => {
        if (!orderNumber) {
            return null;
        }
        const { baseUrl, apiKey } = await shop.getConnection('shop');
        const response = await axios.post<{ number: string; status: string }>(
            `${baseUrl}/shop-order`,
            { orderNumber },
            { headers: { 'api-key': apiKey } }
        );
        return response.data;
    }
);

const aktorTriageResult = createAktorFunction(
    'aktorTriageResult',
    (result: { category: string; summary: string; orderSummary: { number: string; status: string } | null }) => result
);

// Compositions: wire functions into a graph
const aktorClassify = createAktorComposition('aktorClassify', ({ mail }: { mail: Aktor<string> }) =>
    aktorAICall({
        messages: aktorMessagesFromSystemAndUser({
            system: aktorConst('Reply with one word: complaint, question, or order-change.'),
            user: mail,
        }),
        providerModel: aktorAISettingProviderModel(),
    })
);

const aktorSummarize = createAktorComposition('aktorSummarize', ({ mail }: { mail: Aktor<string> }) =>
    aktorAICall({
        messages: aktorMessagesFromSystemAndUser({ system: aktorConst('Summarize in one sentence.'), user: mail }),
        providerModel: aktorAISettingProviderModel(),
    })
);

const aktorTriage = createAktorComposition('aktorTriage', ({ mail }: { mail: Aktor<string> }) => {
    const category = aktorClassify({ mail }); // LLM call
    const summary = aktorSummarize({ mail }); // LLM call
    const orderNumber = aktorFindOrderNumber({ mail }); // plain TypeScript
    const orderSummary = aktorLookupOrder({ orderNumber }); // async, calls the shop connector
    return aktorTriageResult({ category, summary, orderSummary });
});

// A REST endpoint, with its OpenAPI spec generated from the schemas
registerReaktorDefinition({
    reaktorDefinitionId: 'triage-mail',
    label: 'Triage a support mail',
    description: 'Classifies the mail, summarizes it, and looks up the order.',
    aktor: aktorTriage,
    inputSchema: z.object({
        mail: z.string().describe('[label:Support mail][textarea]The mail as the customer wrote it').openapi({
            example:
                'Hello, the coffee machine from order ORD-104233 makes a loud noise when it heats up. Can you help? Best, Anna',
        }),
    }),
    outputSchema: z.object({
        category: z.string(),
        summary: z.string(),
        orderSummary: z.object({ number: z.string(), status: z.string() }).nullable(),
    }),
});
```

The shop is a [Reaktor](#terms "A REST endpoint of an app, built from Aktors, with an OpenAPI spec from its Zod schemas") too: every Reaktor is an API.

`FakeShop.reaktor.ts`

```typescript
import { z } from 'zod';
import { createAktorComposition, createAktorFunction, registerReaktorDefinition } from '@operaide/aktor';
import type { Aktor } from '@operaide/aktor';

// Stands in for a real shop: any REST API works.
const aktorFindOrder = createAktorFunction('aktorFindOrder', async ({ orderNumber }: { orderNumber: string }) => {
    await new Promise((resolve) => setTimeout(resolve, 500));
    return { number: orderNumber, status: 'delivered' };
});

const aktorShopOrder = createAktorComposition('aktorShopOrder', ({ orderNumber }: { orderNumber: Aktor<string> }) =>
    aktorFindOrder({ orderNumber })
);

registerReaktorDefinition({
    reaktorDefinitionId: 'shop-order',
    label: 'Fake shop: look up an order',
    description: 'Stands in for a real shop API. Waits half a second and returns the order.',
    aktor: aktorShopOrder,
    inputSchema: z.object({ orderNumber: z.string().openapi({ example: 'ORD-104233' }) }),
    outputSchema: z.object({ number: z.string(), status: z.string() }),
});
```

The graph the platform draws from this code:

![The graph of the triage composition](triage-diagram.jpg)

The OpenAPI spec generated from the schemas:

![The API explorer with POST /reaktors/triage-mail](triage-openapi.jpg)

A run with the example mail:

![The execution form with the support mail and the JSON result](triage-execution.jpg)

</details>

## Where it runs

One Docker container on your infrastructure.

Build with any model you trust with your code. Run on a model that keeps your data in the house: self-hosted on your own hardware, or hosted in Germany. Because the architecture is decided in advance, building does not need a frontier model either.

Development: yes. Production: no. The Community Edition is licensed for building, testing and demos, paid client work included. Running a business on it needs a paid license.

## Architecture decisions

- **TypeScript, not a language of our own.** Your editor knows it, and so does every model.
- **An app is the unit.** It brings its own UI, its own roles and its own database, and runs in its own process. It is installed, updated and removed as one package.
- **Functions and compositions. That is all.** Functions do the work, compositions wire them into a graph. Parallelism, traces and diagrams come from the graph.
- **Every [Reaktor](#terms "A REST endpoint of an app, built from Aktors, with an OpenAPI spec from its Zod schemas") is an API.** REST, with an OpenAPI spec generated from its schemas. Other apps, other Reaktors and your existing systems call it the same way.
- **The IDE lives inside the platform,** not the platform inside an IDE.
- **git and npm as the transport.** No package format of our own.
- **Best practices live in the workspace,** not in the agent. Any agent builds the same way.

## How to get in

1. **Request access** with your email and GitHub handle.
2. **We invite you** to this organization: the repositories, the handbook, the Discord server, and €5 of credit for models hosted in Germany from our partner Noirdoc.
3. **One container, one description.** `docker compose up`, add a model, switch on the App Builder, describe your first app.

This organization goes public after the preview, with everything written in it. Write as if it were public today.

<p align="center">
  <a href="https://community-edition.operaide.ai/apply"><strong>Request access</strong></a><br>
  <sub>Use your personal GitHub account: managed enterprise accounts cannot be invited</sub>
</p>

## Terms

- **Aktor**: one step of a workflow, either a function or a composition of other Aktors.
- **Composition**: an Aktor that wires other Aktors into a graph. It is declarative: it describes the graph and does not run it.
- **Reaktor**: a REST endpoint of an app, built from Aktors. Its Zod input and output schemas generate the endpoint and its OpenAPI spec.
- **-ilities**: what every app needs besides its purpose: deployment, login, roles, audit, tracing.

<p align="center"><sub>Operaide is a brand of objective partner AG · <a href="https://operaide.ai/legals/imprint">Imprint</a> · <a href="https://operaide.ai/legals/privacy-policy">Privacy Policy</a> · <a href="https://operaide.ai/contact">Contact</a></sub></p>
