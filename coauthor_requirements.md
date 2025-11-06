## Coauthor-Kit AI: A Spec-Driven Engine for AI-Native Books

Instead of writing a manuscript from scratch, authors **co-create a `ManuSpec`** (a "book specification") through a structured conversation with an AI. This spec, not a traditional manuscript, becomes the single source of truth that AI agents use to build, test, and maintain the content. It includes examples of ManuSpec:

https://chatgpt.com/share/690c3dd2-de08-8001-b57a-564aaeffa17f

https://gemini.google.com/share/361b1a7a4809

i think the best approach is to first generate a Markdown-formatted manuspec (not in yaml or json) for our book and then use that as the foundation for our Docusaurus files. the manuspec should be in markdown not in yaml or json.

A Markdown-first ManuSpec is the most author-friendly path: you get readable diffs, zero friction in PRs, and a single file that both humans and the compiler can understand. Then your build step can emit Docusaurus MDX + sidebars from that Markdown. Below is a practical Markdown DSL (“SpecMD”) you can adopt today, plus ready-to-use templates for BOOK.manuspec.md and CHAPTER.manuspec.md, and the exact mapping your compiler should follow.



### 1. The Vision: From Manual Writing to Conversational Specs

Technical writing is broken. Authors wrestle with static, monolithic documents, while readers are left with a one-size-fits-all experience.

Coauthor-Kit AI is a new ecosystem for creating and experiencing technical nonfiction, built on one core idea: **separating authorial intent from the final product.**

Instead of writing a manuscript from scratch, authors **co-create a `ManuSpec`** (a "book specification") through a structured conversation with an AI. This spec, not a traditional manuscript, becomes the single source of truth that AI agents use to build, test, and maintain the content.

This ecosystem is built on two distinct components:

1.  **The Coauthor-Kit Engine:** An open-source, local-first toolchain that guides the author in creating the `ManuSpec` and then "compiles" that spec into a beautiful, deployable book.
2.  **The Book-Hub Platform:** An optional, cloud-hosted platform that "activates" the compiled book with live, AI-native features for readers.

---

### 2. For Authors: The Coauthor-Kit Engine

This is the local-first, open-source CLI and toolchain (e.g., `coauthor-cli`) built for a modern, Git-native workflow. It redefines the authoring process in two parts: conversational creation and AI-powered compilation.

#### Part 1: The "Spec-Kit" Workflow (Creating the ManuSpec)

The authoring process no longer starts with a blank page. It starts with a guided, conversational process to build your `ManuSpec`.

Using an integrated tool, the author **converses with an AI assistant (powered by Claude Code)**. This conversation is structured by **GitHub Spec-Kit**, enforcing a rigorous methodology to define the book's DNA:

1.  **Constitution:** Defining the high-level goals, immutable rules, voice, and core identity of the book.
2.  **Specs:** Breaking down the constitution into concrete, testable specifications for audience, scope, and technical depth.
3.  **Plan:** Generating a detailed chapter-and-lesson-level structure based on the specs.
4.  **Task:** Breaking the plan into discrete implementation tasks for the AI agents.
5.  **Implement:** Executing the tasks to generate the content.

The **`ManuSpec`** is the final *artifact* of this conversation—a rich, structured file that encodes all these decisions. This spec is what you commit to Git.

#### Part 2: The "Compiler" Workflow (Executing the ManuSpec)

Once your `ManuSpec` is committed, the Coauthor-Kit Engine's "compiler" takes over. The Engine reads your spec and orchestrates a team of specialized sub-agents to execute it. These agents, which integrate with powerful tools like Gemini CLI and Claude Code, include:

* **Outline Architects:** To structure the content graph.
* **Evidence Gatherers:** To find and integrate data and citations.
* **Code & Math Verifiers:** To ensure technical accuracy.
* **Narrative Stylists:** To enforce your specified voice.
* **Compliance Checkers:** To validate the output against your spec's acceptance criteria.

#### Git-Native Workflow

The entire process fits naturally into your existing developer workflow:

* **Reviewable Diffs:** Every AI-generated change is a clear `diff` you can review, accept, or reject.
* **Deterministic Drafts:** Re-run the engine to get a consistent draft from your spec.
* **Version Control:** Your `ManuSpec` is versioned in Git, just like code.

The final output is a beautiful, enhanced static site. We use Docusaurus as our presentation layer, "compiling" your content into MDX files, component definitions, and graph data that can be deployed to any static host (like GitHub Pages or Netlify) for free.

---

### 3. For Readers: The Book-Hub Platform Experience

When an author chooses to deploy their book to our optional, cloud-hosted **Book-Hub Platform**, the static content "wakes up" with a suite of AI-native features.

* **🧠 Always-on Co-Teacher:** An AI agent that has read the entire book, knows what page the reader is on, and can answer questions, provide summaries, and quiz them on the material.
* **💬 Socratic Mode:** A co-teacher mode that guides the reader to the answer with questions, hints, and reflection prompts instead of just giving the solution.
* **🌱 Personalized & Adaptive Learning:** Text, examples, and difficulty adjust in real-time to the reader's background, goals, and struggles.
* **🔬 Branching & Sandboxed Simulations:** Chapters come alive with runnable code cells, micro-simulations, and "what-if" scenario knobs that generate new text or diagrams on the fly.
* **🎬 Multimodal by Default:** Inline generated video overviews, interactive diagrams, and quick-explainers, all without leaving the page.
* **🤝 Social & Multiplayer Reading:** A "Cohort Mode" lets a class or team read together with shared annotations, live polls, and agent-moderated discussions.
* **📈 Integrated Assessments:** Spaced repetition, goal tracking, and assessments are core features, not add-ons.
* **🔒 Privacy-Aware:** The platform uses a hybrid model. Heavy lifting is done by cloud agents, while on-device models (like Gemini Nano) can be used for privacy-sensitive tasks like summarizing a user's personal notes.

---

### 4. Our Two-Product Architecture

This entire vision is made possible by a clean separation of concerns.

| Component | **Product 1: The Coauthor-Kit Engine (`coauthor-cli`)** | **Product 2: The Book-Hub Platform (SaaS)** |
| :--- | :--- | :--- |
| **What it is** | A free, open-source, local-first CLI tool. | A paid, cloud-hosted, multi-tenant SaaS platform. |
| **What it does** | Guides authors through `ManuSpec` creation, then "compiles" it into a deployable static site (using Docusaurus for presentation). | Serves the book and powers all real-time, stateful, and AI-native reader features (co-teacher, database, user accounts). |
| **Key Features** | Conversational `ManuSpec` creation (via Spec-Kit/Claude), AI agent orchestration, Git integration, reviewable diffs, static site generation. | AI co-teacher, adaptive content logic, user auth, cohort mode, analytics, and live updates. |
| **Hosting** | Runs locally or in a CI/CD pipeline. Deploys anywhere. | Deployed to our managed, scalable cloud infrastructure. |

This architecture also simplifies our "hubs." The **Book-Hub Platform** is the single backend that provides different *views* (dashboards) based on user roles:

* **Publisher Dashboard:** For analytics and book management.
* **Teacher Dashboard:** For managing cohorts and assignments.
* **Author Dashboard:** For communicating with readers.

---

### 5. Our Business Model: Free Tools, Paid Platform

Our business model is as straightforward as our architecture.

* **✅ What's Free: The Coauthor-Kit Engine**
    The entire toolchain to *create* content is **100% free and open-source, forever.** Authors can use our `coauthor-cli`, the conversational Spec-Kit workflow, and deploy their static books to GitHub Pages, Netlify, or their own servers at no cost. This builds our community and drives adoption.

* **💰 What's Paid: The Book-Hub Platform**
    We only charge for the high-value, high-cost, server-side features. Our revenue comes from authors and publishers who *choose* to deploy to our platform to activate the "live" AI features.

#### Tiers

1.  **Free Tier:** Use the `coauthor-cli` to build and host static books yourself.
2.  **Pro Tier (For Individuals):** Deploy your book to our Book-Hub to activate the AI co-teacher and get reader analytics.
3.  **Team Tier (For Companies & Schools):** Everything in Pro, plus cohort mode, teacher/publisher dashboards, and SSO.

Monetization is based on usage (e.g., AI interactions, database use) after a generous free tier, as well as per-seat licenses for Team/Cohort features.

# Should the Spec be Generated First?

Based on your entire "Coauthor-Kit AI" architecture, you should **definitely generate the YAML files first**, and then use those files to build the book.

The core strength of your product is the **ManuSpec**—the structured specification (in YAML format) that dictates the book's creation.

---

## 🛠️ Why Generate the YAML (`ManuSpec`) First?

Generating the `ManuSpec.yml` file is not just a technical step; it is the **fundamental value proposition** of your product.

| Approach | Advantage for Coauthor-Kit AI |
| :--- | :--- |
| **Generate YAML First** | **Establishes Control and Determinism:** The YAML is the single source of truth. It contains all the author's critical decisions (constitution, specs, voice, acceptance criteria). This is necessary for the next step. |
| **Generate Book Directly** | **Bypasses the Core Innovation:** If you generate the book directly, you lose the ability to apply version control, audit the inputs, and enforce the constraints defined by the Spec-Kit methodology. The output becomes a "black box." |

### 1. **Enforcing Spec-Driven Methodology**

The entire process is predicated on the **GitHub Spec-Kit** workflow (Constitution $\rightarrow$ Specs $\rightarrow$ Plan $\rightarrow$ Task $\rightarrow$ Implement).

* The YAML file is the output of the first four steps (Constitution, Specs, Plan, Task).
* The actual book content is the output of the fifth step (Implement).

You need the structured, versioned, and auditable **plan** (`ManuSpec.yml`) *before* you execute the potentially expensive and variable content generation.

### 2. **Enabling the Git Workflow**

You highlighted that your product "fits naturally into **Git workflows** and provides **reviewable diffs** for every pass."

* The author **reviews and commits the `ManuSpec.yml`** (the plan) to Git. This ensures the *author's intent* is versioned.
* Then, the Coauthor-Kit Engine generates the final book content based on that committed plan.
* If the generated book content is also committed, the author can review the diffs between two versions of the *book* and trace any changes back to the source `ManuSpec` that caused them.

### 3. **Separating Author Intent from AI Execution**

The YAML file is the clean break between the **Human/Spec-Kit Conversation** and the **AI Compilation Engine**.

* **Human/Claude Code's Job:** Create, refine, and commit the precise instructions in the YAML.
* **Sub-Agents' Job:** Read the YAML and deterministically execute the implementation steps.

This separation is crucial for debugging, maintenance, and providing the **Live Updates with Versioning** feature you planned.

---

**Conclusion:**

The conversational process with Claude Code should be engineered to **produce and refine the `ManuSpec.yml` file.** This YAML file then acts as the **blueprint** that the sub-agents read to *build the actual book content*.

