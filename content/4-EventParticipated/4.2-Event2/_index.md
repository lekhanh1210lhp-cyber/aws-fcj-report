---
title: "Event 2"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---

# Report: “AI-Driven Development Lifecycle: Reimagining Software Engineering”

### Event Objectives

- Understand how AI can automate and optimize stages of the Software Development Lifecycle (SDLC).
- Embrace the philosophy of AI augmenting humans rather than replacing them in the software development process.
- Observe how tools like Amazon Q and other AI assistants support developers from ideation and code generation to infrastructure deployment (IaC).
- Learn about the emerging trend of “AI-first development,” where AI becomes a natural part of future dev workflows.

### Speakers

- **Toan Huynh**
- **My Nguyen**

### Highlights

#### Challenges of programming with AI

The introduction covered the limitations and challenges of applying AI to programming:

- AI still struggles with projects that require deep domain knowledge and complex business logic.
- Developers can have limited control over generated code when prompts and scope are not well-defined.
- The quality of generated code depends heavily on the prompt and context provided to the model.

This motivates the AI-DLC approach: creating a structured process to help AI and humans collaborate more effectively.

#### How AI is changing software development

This section analyzed how AI is transforming the software industry:

- AI assists code generation, technical documentation, API design, and automated testing.
- Developers shift roles from “code writers” to “AI orchestrators” who guide, evaluate, and refine AI outputs.
- Tools like Amazon Q, GitHub Copilot and ChatGPT for Developers become central parts of modern development workflows.

#### 🔹 What is AI-DLC

AI-Driven Development Lifecycle (AI-DLC) is an AI-augmented software development approach where each stage is designed to provide AI with specific context and goals to produce more accurate results.

🟧 Inception

1. Build Context on Existing Code – feed AI the current codebase so it understands project structure.
2. Elaborate Intent with User Stories – developers describe requirements via user stories to clarify goals.
3. Plan with Units of Work – break work into small units the AI can execute and generate code for.

🟦 Construction

4. Domain Model (Component Model) – build domain models or architecture diagrams.
5. Generate Code & Test – AI generates code and tests based on the plan.
6. Add Architectural Components – add API layers, data layers, logging, and security components.
7. Deploy with IaC & Tests – automate deployment using Infrastructure as Code and integration tests.

_🔁 Each stage provides richer context for the next, helping AI produce increasingly accurate outputs._

#### Core Concepts

1. Context Awareness – AI needs clear context about code, requirements, and domain to work well.
2. Collaborative Generation – humans and AI collaborate: AI generates code, humans direct and verify outputs.
3. Continuous Refinement – iterative cycles to refine outputs and improve quality.

#### Mob Elaboration

Mob Elaboration is a collaborative method for elaborating intents:

- Multiple participants contribute user stories, questions, and additional context for the AI.
- It helps AI gain deeper understanding of domain, goals, and complex logic.
- This approach reduces the risk of misunderstandings—especially in large or cross-domain teams.

#### The 5-Stage Sequential Process of AI-DLC

AI-DLC runs through 5 phases:

1. Inception – understand requirements and analyze the system.
2. Construction – create domain models and initial structure.
3. Generation – automated code generation.
4. Testing – automated unit and integration testing.
5. Deployment – deploy applications with IaC and CI/CD pipelines.

Each loop improves the AI's outputs through incremental learning and feedback.

#### Demo 1 – Interactive AI-DLC experience with Amazon Q

The demo showcased AI-DLC in practice with a small project:

- Start from a simple idea and turn it into a user story describing business requirements.
- AI helps split tasks into Units of Work and plans implementation details for each module.
- Attendees interact with AI using questions, checkboxes, and logical conditions to clarify scope.
- AI generates code, tests, project structure, and executes trial deployments.
- The demo illustrated smooth collaboration between AI and humans: AI performs repetitive generation while humans steer and make decisions.

#### Introducing Kiro

Philosophy of Kiro

The workshop introduced Kiro, an intelligent development environment built around the idea of “AI-native development” where AI is a core collaborator rather than just a tool.

Kiro’s philosophy emphasizes three points:

1. Deep integration with the development process – AI participates in planning, context management, and impact analysis.
2. Comprehensive project context – Kiro maintains ongoing awareness of project structure so AI can interact with the whole project rather than single files.
3. Intelligent control & collaboration – developers guide AI via contextual commands so each change has clear intent and consistency.

This makes Kiro more than a code generator: it is an ecosystem for collaborative human–AI development.

Project structure in Kiro

Unlike traditional text editors like VSCode or JetBrains, Kiro is an AI-aware workspace with structural awareness.

Its project model includes:

- Context Layer – stores context, domain models, and relationships among modules.
- Task Layer – manages Units of Work tracked and executed by AI.
- AI Agent Layer – agents handle specific tasks (code, tests, refactor, deploy) enabling a multi-agent collaborative model.
- Human-in-the-Loop Control – developers can confirm, modify, or reject AI outputs at any stage.

Kiro therefore becomes an ecosystem for coordinated human–AI development rather than just a code editor.

#### Demo 2: Kiro in practice

In the demonstration, the presenters showed how Kiro implements AI-DLC:

1. User provides a basic business requirement like “build an event management system.”
2. Kiro analyzes intent, creates a domain model, and splits work into user stories.
3. AI generates modules, components, and corresponding test cases.
4. Developers interact with a checkbox-based task control to approve each unit of work.
5. Kiro finally deploys the completed system using IaC and automated tests.

The demo proved AI-DLC is practical: AI, human operators, and processes integrate into a single coherent workflow.

### Event experience

Attending the workshop “AI DLC x Kiro: Reinventing Developer Experience with AI” was highly valuable, clarifying how AI can be deeply embedded into the developer experience and how Kiro’s design offers a fresh approach for developers.

#### Insights from expert speakers

- Speakers presented AI-DLC as a platform that automates many SDLC tasks and supports software development using AI.
- The Kiro introduction gave a perspective on designing an AI-native text editor rather than adding AI plugins to legacy editors.
- I was particularly impressed by Kiro’s philosophy: minimalism, high performance, user-focused experience, and modular extensibility.

#### Practical technical takeaways

- The demo showed how AI-DLC and Kiro can create, refactor, and optimize code efficiently.
- A small starter project was created and managed within Kiro, demonstrating auto-refactoring, test generation and logic analysis.
- Compared to editors like VSCode and Sublime, Kiro stands out for its AI-first architecture and lightweight plugin model that preserves performance.

#### Modern tooling and potential applications

- Experiencing AI-DLC on Kiro highlighted the potential to automate development workflows—especially code generation, documentation, and debugging.
- I saw opportunities to build personal learning and productivity tools that provide smart suggestions and accelerate development.
- Kiro’s modular design inspires approaches to building flexible, maintainable systems.

#### Networking and discussions

- The workshop offered chances to connect with developers, AI researchers, and product designers, deepening my understanding of AI-augmented development.
- Discussions helped me see AI as a creative collaborator, allowing developers to focus more on system logic and architecture.

#### Key lessons

- AI-DLC combined with Kiro is a model for next-generation development tools—AI-first IDEs that deeply integrate AI into the workflow.
- Kiro’s “less is more” philosophy shows that simplicity and performance can deliver a stronger developer experience than overly complex systems.
- I learned that successful AI adoption depends not only on technology but also on the design philosophy and integration approach used in tooling.

#### Sample images from the event
