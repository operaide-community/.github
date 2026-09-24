<p align="center">
  <img src="https://raw.githubusercontent.com/operaide-community/.github/main/profile/operaide-icon.svg" alt="Operaide" width="120">
</p>

<h1 align="center">The Operaide Community Edition</h1>

<p align="center"><strong>100 developers · 8 weeks · Building AI together</strong></p>

<p align="center">Applications open until September 30, 2026</p>

We are looking for 100 developers to test Operaide exclusively for 8 weeks, build their own AI applications, and evolve the platform together with our team. You get access to the GitHub repositories of the Community Edition, build your own projects, and share your feedback directly with us.

<p align="center">
  <a href="https://community-edition.operaide.ai/apply"><strong>Apply for one of the 100 spots</strong></a><br>
  <sub>Under a minute · free</sub>
</p>


![Operaide Studio with the Operaide Code agent](https://raw.githubusercontent.com/operaide-community/.github/main/profile/framework.png)

## What is Operaide?

Operaide is a platform for building and running AI applications. Authentication, multi-tenancy, roles, audit, database, vector store, and deployment are platform primitives. Your code never rebuilds them, it solves the business problem. Workflows are typed TypeScript, not drag-and-drop.

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

`summary` and `sentiment` both depend on `text` and not on each other, so the engine runs the two LLM calls concurrently. You wrote no `Promise.all`.

**Framework: what you build with.** You compose workflows from small typed steps with explicit inputs and outputs. The platform runs them and parallelises whatever is independent.

- Every workflow becomes a REST endpoint with an OpenAPI specification
- Apps ship their own UI: a review queue, a dashboard, a data-entry screen
- Operaide Studio in the browser. Claude Code, Codex, and Operaide Code know the patterns

**Platform: where it runs.** One Docker container behind the reverse proxy of your choice, database included. Larger installations can split components. Simplicity comes first.

- Every execution recorded, every LLM call tracked for usage
- Audit trail throughout
- Cloud, VPC, or on-premises: your infrastructure, your data

![Operaide Platform: the diagram of a multi-agent application](https://raw.githubusercontent.com/operaide-community/.github/main/profile/platform.png)

## What is the Community Edition?

The complete Operaide platform, Framework and Platform, with no gated features. You host it yourself with Docker: locally on your machine or in any cloud.

| Included | The one limitation | And after the 8 weeks? |
| --- | --- | --- |
| Framework and Platform, the full feature set. No gated features, no time-limited access. Self-hosted via Docker, locally or in any cloud. Installed in a few minutes. | Development and testing: yes. Production use: no. Build, test, and experiment without limits. For productive use in the enterprise there is the Enterprise Edition. | You keep the Community Edition, it does not switch off. Afterwards we publish the official Community Edition for everyone. The 8 weeks with 100 developers are the format in which we finish building it together. |

## What do you get?

- **Exclusive access** to the Operaide Community Edition, and to its repositories in this organization.
- **8 weeks of practice.** Time to build your own projects and really put Operaide to the test.
- **Direct exchange.** Feedback and exchange with the Operaide team on our Discord server.
- **Influence on the roadmap.** Your experience helps us evolve Operaide.
- **Publish what you build.** Push your apps and templates to npm, they show up in the Operaide App Store, ready for anyone to use. The first community apps in the store will come out of this round.
- **Regular updates.** We ship new features continuously, built straight from the community's feedback. You see your suggestions land in the product.

## Who are we looking for?

You are a fit if you:

- Build software or your own technical projects
- Can read and review TypeScript. You do not have to type every line yourself
- Are interested in AI, LLMs, or AI agents
- Enjoy trying out new technologies
- Want to build something with Operaide over the 8 weeks
- Are willing to give honest feedback

AI expert or beginner? Both are welcome.

## How it works

1. **Apply.** Through the form. It takes under a minute. By September 30, 2026.
2. **We select 100.** We are looking for developers who are curious, like trying things out, and want to realise their own ideas.
3. **Build with Operaide.** You get access to the Community Edition, to our Discord server, and 8 weeks to build with it.
4. **Shape what's next.** Share your experience directly with us and help make Operaide better.

The organization is private during the preview and goes public afterwards. Everything you write there, issues and pull requests included, becomes public then. Write as if it were public today.

<p align="center">
  <a href="https://community-edition.operaide.ai/apply"><strong>Apply for one of the 100 spots</strong></a><br>
  <sub>Apply by September 30, 2026 · free</sub>
</p>

<p align="center"><sub>Operaide is a brand of objective partner AG · <a href="https://operaide.ai/legals/imprint">Imprint</a> · <a href="https://operaide.ai/legals/privacy-policy">Privacy Policy</a> · <a href="https://operaide.ai/contact">Contact</a></sub></p>
