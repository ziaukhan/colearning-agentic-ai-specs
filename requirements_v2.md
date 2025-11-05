Coauthor-Kit AI

Coauthor-Kit AI is the spec-aware coauthoring engine that reads your ManuSpec and executes it with specialized subagents —outline architects, evidence gatherers, code and math verifiers, narrative stylists, and compliance checkers—each optimized for technical nonfiction. It runs locally and is stored on GitHub, fits naturally into Git workflows, and provides reviewable diffs for every pass. From outline to publish-ready copy, it enforces your constraints, preserves voice, and scales effort across chapters while keeping you in control of constitution, specs, decisions, sources, and standards. It uses Docusaurus, GitHub, Claude Code, Gemini CLI, and Spec-Kit PlusWe want to thin fork Docusaurus with a objective of building optimised spec driven documentation websites and digital books with AI-native experiences quickly. It will be a thin fork because it will be using a fork of github spec-kit and claude code and Gemini CLI for spec driven authoring. 

Where possible we will use  Gemini in Chrome Features.


Features for the readers

* The content is a graph with Parts, Chapters, and Lessons.
* Always-on co-teacher. Each book ships with an agent that answers questions, quizzes you, and suggests detours (“You struggled with Chapter 2—want a 10-minute primer?”). The co-teacher will always know what the reader is reading right now, so that it can help on that material if required. 
* Co-teacher Socratic style mode. The co-teacher will have mode where it will use the Socratic-style approach: it guides you with questions, hints, and reflection prompts instead of just handing over the answer.
* Personalised and Adaptive learning Mode. Text, examples, and difficulty adjust in real time to readers background and goals, if the reader is unable to understand the original generic material.
* Branching & sandboxed simulation. Chapters include runnable widgets: code cells, micro-sims, data toys, and scenario knobs that produce new text, diagrams, or explanations on the fly.
* Multimodal by default. Inline generated video overviews and explainers, generated diagrams, quick AR callouts —without leaving the page.
* Quick AR callouts = tiny, tap-to-view augmented-reality 3D moments you sprinkle into a chapter.
* Social & multiplayer reading. “Cohort mode” lets a class or team read together with shared annotations, live polls, and agent-moderated discussions.
* Assessments at the end of each chapter. Spaced repetition, assessments, and goal tracking are core features, not add-ons.
* Privacy-aware. On-device models handle sensitive notes; cloud agents handle heavy lifting. Your learning profile is yours.




Features for authors & publishers
* The author does not write a manuscript but a ManuSpec. ManuSpec is the single source of truth for AI-native publishing—a structured specification that encodes objectives, audience, scope, voice, references, and testable acceptance criteria for every part, chapter and lesson/section. Treating the book as a build artifact, ManuSpec enables deterministic drafts, consistent revisions, and precise updates. It integrates seamlessly with version control, enables automated quality gates, and orchestrates agents to constitution, spec, plan, draft, fact-check, and refine. With ManuSpec, authors gain repeatability, teams gain alignment, and readers get books that are accurate, coherent, and maintainable.
* From manuscript to content graph. The Generic Initial Books become nodes (concepts, skills, examples) with relationships and prerequisites. The agent composes a path through that graph per reader when personalising the generic book on request.
* Live updates with versioning. Facts and data can update without breaking citations; readers can pin versions for stable coursework.
* New business models. Book-as-a-service (updates + analytics) and cohort licenses for classrooms


Development Notes

* Coauthor-Kit AI require a heavy-weight, stateful backend, user authentication, a database, and complex real-time server logic.
* We are building a complex web application that might use Docusaurus's frontend components and Markdown rendering.
* We will try to use where possible Gemini in Chrome (Gemini Nano) API.
* Ww have two very different products mixed together. 
* The Authoring Engine: The "ManuSpec" compiler, sub-agents, and Git tools. This (mostly) runs locally or in a CI/CD pipeline. 
* The Reading Platform: The hosted website with the co-teacher, multiplayer, and adaptive content. This is 100% a cloud service. These two systems need to be architecturally distinct, even if they are sold as one product. The "build artifact" from the local engine is deployed to the cloud platform.
* The reading platform can deployed in many ways including GitHub Pages, Netlify, and other static site hosts like Render. GitHub Pages is a popular and easy option that integrates well with Docusaurus, but you can deploy to any service that hosts static sites. 

 
The “Five-Product" Architecture

Clearly define your architecture as five distinct systems that work together:
* Product 1: The Author's Engine (e.g., coauthor-cli)
    * What it is: A local CLI, Git hooks, and Spec-Kit and Claude Code extension.
    * What it does: Reads ManuSpec.yml, orchestrates AI agents (using "Spec-Kit Plus" as the brain) via their APIs (Gemini, Claude), and "compiles" the source content into a deployable "build package" (e.g., MDX, component definitions, and graph data).
    * Key Feature: The diff review and Git integration.
* Product 2: The Reader's Platform (e.g., “Book-Hub")
    * What it is: A cloud-hosted SaaS platform.
    * What it does: You deploy your "build package" to this platform (Can use GitHub Pages). It serves the content and powers all the AI-native reader features (the co-teacher agent, user accounts, adaptive content logic, and multiplayer discussions).
    * This also requires a heavy-weight, stateful, and multi-tenant backend, with user authentication, a database, and complex real-time server logic. Everything will be open source and free except for this back-end. We will change per interaction after a free tier.
* Product 3: The Publisher Dashboard (e.g., “Publisher-Hub")
    * What it is: A cloud-hosted SaaS platform.
    * What it does: It uses the multi-tenant backend APIs to show analytics.
* Product 4: The Cohort Human Teacher Dashboard (e.g., “Cohort/Class-Hub")
    * What it is: A cloud-hosted SaaS platform.
    * What it does: It uses the multi-tenant backend APIs to allow cohort human teachers to communicate with, answer questions, and give assignments to the cohort/class students.  
* Product 5: The Author Dashboard
    * What it is: A cloud-hosted SaaS platform.
    * What it does: It uses the multi-tenant backend APIs to allow human authors to communicate with, answer questions from the readers.  


Business Model

* We only charge for the heavy-weight, stateful, and multi-tenant backend, with user authentication, a database, and complex real-time server logic. Everything will be open source and free except for this back-end APIs. We will only charge for these API calls after a free tier.
* Reader Platform APIs: This is our monetisation.
* Pro Tier: For individual authors to host their "live books”, with analytics.
* Team Tier: For companies to host their internal/external documentation with cohort features, analytics, and Single Sign-On.( single set of credentials (like a username and password) to gain access to multiple different documentation and books )




