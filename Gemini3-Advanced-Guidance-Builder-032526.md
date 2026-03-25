Technical Specification: SmartMed Review 3.0
1. Executive Summary
1.1 Purpose
The SmartMed Review 3.0 system is an advanced, AI-powered web application designed specifically to streamline, enhance, and automate the Medical Device Regulatory Affairs review process. By leveraging state-of-the-art Large Language Models (LLMs) via the Google Gemini API, the system assists regulatory professionals in ingesting complex medical device documentation, triaging submissions, generating regulatory guidance, and authoring comprehensive review reports.
1.2 Scope
This technical specification outlines the architecture, component design, data flow, AI integration strategy, and security considerations for the SmartMed Review 3.0 frontend application. The application is built as a Single Page Application (SPA) using React, TypeScript, Vite, and Tailwind CSS, operating primarily within the client's browser to ensure data privacy and rapid interaction, while interfacing with external AI APIs for heavy cognitive processing.
1.3 Target Audience
This document is intended for software engineers, frontend developers, UI/UX designers, regulatory affairs domain experts, and project stakeholders involved in the development, maintenance, and scaling of the SmartMed Review platform.
2. System Architecture Overview
2.1 High-Level Architecture
SmartMed Review 3.0 employs a client-heavy, serverless architecture. The core application logic, state management, and UI rendering occur entirely within the user's browser. The application communicates directly with the Google Gemini API for natural language processing, document summarization, and guidance generation.
The architecture is divided into three primary layers:
Presentation Layer: React components styled with Tailwind CSS and Radix UI primitives.
State & Logic Layer: Zustand for global state management, encapsulating business logic and data transformations.
Integration Layer: API clients for communicating with the Google Gemini API and local file system APIs for document ingestion.
2.2 Technology Stack
Core Framework: React 19.0, TypeScript 5.8
Build Tool: Vite 6.2
Styling: Tailwind CSS 4.1, Radix UI (Headless components), clsx, tailwind-merge
State Management: Zustand 5.0
AI Integration: @google/genai 1.29.0
Document Processing: pdfjs-dist 5.5, react-pdf 10.4
Markdown & Visualization: react-markdown 10.1, remark-gfm, mermaid 11.13
Animations: framer-motion, motion 12.23
3. Core Modules & Component Specifications
The application is structured around a tab-based navigation system, managed by the App.tsx and Layout.tsx components. Each tab corresponds to a specific phase of the regulatory review workflow.
3.1 Document Ingestion Module (DocIngestion.tsx)
Purpose: To securely load, parse, and extract text from complex medical device submissions (e.g., 510(k), PMA, MDR documents) in PDF format.
Technical Details:
File Handling: Utilizes HTML5 File API for drag-and-drop and file selection.
PDF Parsing: Integrates react-pdf and pdfjs-dist to render PDF pages and extract raw text layers.
Chunking Strategy: Implements a text chunking algorithm to break down massive regulatory documents into manageable segments that fit within the Gemini API's context window.
Metadata Extraction: Automatically attempts to identify document types, submission dates, and applicant information using regex and lightweight initial AI passes.
3.2 Guidance Generator Module (GuidanceGenerator.tsx)
Purpose: To synthesize extracted document text with known FDA/MDR regulations and generate actionable review guidance.
Technical Details:
Prompt Engineering: Constructs dynamic prompts combining the user's specific query, the ingested document context, and a predefined system persona (e.g., "Expert FDA Regulatory Reviewer").
Streaming Responses: Utilizes the Gemini API's streaming capabilities (generateContentStream) to provide real-time feedback to the user, reducing perceived latency during complex generations.
Context Management: Maintains a sliding window of conversation history and document context to ensure the AI's responses remain relevant to the current review session.
3.3 Markdown Editor Module (MarkdownEditor.tsx)
Purpose: To provide a robust authoring environment for regulatory professionals to refine AI-generated guidance and draft final review reports.
Technical Details:
Rich Text Rendering: Uses react-markdown with remark-gfm to support GitHub Flavored Markdown, including tables, task lists, and strikethroughs, which are essential for regulatory checklists.
Diagram Support: Integrates mermaid to allow users to define and render complex flowcharts (e.g., decision trees, manufacturing processes) directly within their markdown reports.
Sync Scrolling: Implements synchronized scrolling between the raw markdown input pane and the rendered preview pane.
3.4 Submission Triage & Review Report (Placeholders.tsx)
Purpose: Currently architected as placeholders for future expansion.
Submission Triage: Will analyze incoming documents against acceptance criteria (e.g., RTA - Refuse to Accept policy) to determine if a submission is complete enough for substantive review.
Review Report: Will aggregate findings from the Guidance Generator and Markdown Editor into a standardized, exportable format (e.g., PDF, DOCX) conforming to agency templates.
3.5 Telemetry Terminal (TelemetryTerminal.tsx)
Purpose: To provide developers and power users with real-time insights into the application's internal state, API interactions, and performance metrics.
Technical Details:
Log Interception: Hooks into the global state and API clients to log token usage, latency, and error rates.
Visual Console: Renders a mock-terminal interface using monospace fonts and color-coded log levels (INFO, WARN, ERROR).
3.6 Settings & Configuration (Settings.tsx)
Purpose: To manage user preferences, API keys, and application theming.
Technical Details:
Theme Engine: Interacts with the CSS variables defined in index.css. Supports 10 distinct Pantone-inspired themes (e.g., Classic Blue, Emerald, Living Coral) and toggles between light and dark modes by manipulating the .dark class on the root HTML element.
Secure Storage: Manages the input and local storage of the Gemini API key. (Note: In the AI Studio environment, this is often injected via process.env, but the UI provides a fallback/override mechanism).
4. Data Flow and State Management
4.1 Zustand Store (store.ts)
The application relies on Zustand for global state management, providing a lightweight, hook-based API.
Predicted Store Structure:
code
TypeScript
interface AppState {
  // Document State
  documents: UploadedDocument[];
  activeDocumentId: string | null;
  addDocument: (doc: UploadedDocument) => void;
  
  // AI & Context State
  extractedText: string;
  setExtractedText: (text: string) => void;
  chatHistory: ChatMessage[];
  addChatMessage: (msg: ChatMessage) => void;
  
  // UI State
  theme: string;
  setTheme: (theme: string) => void;
  isDarkMode: boolean;
  toggleDarkMode: () => void;
  
  // Config State
  apiKey: string;
  setApiKey: (key: string) => void;
}
4.2 Data Persistence
To ensure continuity across browser sessions, critical non-sensitive state (like theme preferences and UI layouts) is persisted to localStorage. Sensitive data, such as the actual contents of medical device submissions, remains strictly in memory (sessionStorage or React state) to comply with data privacy standards and is cleared upon application exit or page refresh.
5. AI Integration Strategy
5.1 Gemini API Implementation
The application uses the @google/genai SDK. All calls to the Gemini API are executed client-side.
Model Selection:
Standard Tasks (Summarization, Extraction): gemini-3-flash-preview is utilized for its speed and cost-effectiveness.
Complex Reasoning (Regulatory Analysis, Triage): gemini-3.1-pro-preview is employed when deep contextual understanding of regulatory frameworks (e.g., 21 CFR Part 820, ISO 13485) is required.
5.2 Prompt Architecture
Prompts are dynamically constructed using a template system:
System Instruction: Defines the persona ("You are a Senior FDA 510(k) Reviewer...").
Context Injection: Injects the relevant chunks of the ingested PDF document.
Task Definition: The specific user query or automated task (e.g., "Identify all predicate devices mentioned in this section").
Output Formatting: Enforces JSON schema or specific Markdown structures for predictable parsing by the UI.
5.3 Token Management
Given the massive size of regulatory submissions, the application implements a token estimation utility. If a document exceeds the model's context window, the system employs a Map-Reduce summarization strategy or allows the user to query specific document sections interactively.
6. User Interface & Experience (UI/UX)
6.1 Design System
The UI is built on a foundation of Tailwind CSS and Radix UI primitives, ensuring high accessibility (WAI-ARIA compliance) and robust keyboard navigation.
6.2 Theming Engine
The index.css file defines a sophisticated theming system using CSS variables. It supports a base light/dark mode and multiple color palettes based on Pantone colors (e.g., theme-classic-blue, theme-very-peri). The UI dynamically applies these classes to the <body> or root <div> to instantly switch the application's visual identity.
6.3 Animation Strategy
framer-motion and motion are used to provide fluid transitions between the main application tabs (Ingestion -> Generator -> Editor). This spatial awareness helps users maintain mental context as they move through the complex review workflow.
7. Security & Compliance Considerations
7.1 Client-Side Processing
By processing documents and communicating with the AI API directly from the browser, the application avoids storing sensitive proprietary medical device data on intermediate backend servers.
7.2 API Key Security
The Gemini API key is required to be provided by the user or injected via the environment. The application must ensure that this key is never logged to the Telemetry Terminal or exposed in URL parameters.
7.3 Regulatory Context
While the application assists in regulatory review, it is designed as a decision-support tool. The UI must clearly delineate AI-generated content and require human-in-the-loop verification before any review report is finalized, aligning with software-as-a-medical-device (SaMD) and general quality system guidelines.
8. Deployment and Infrastructure
8.1 Build Pipeline
The application is bundled using Vite (vite.config.ts). The build process compiles TypeScript, processes Tailwind CSS, and optimizes assets for production deployment.
8.2 Hosting
The application is designed to be hosted on serverless container platforms like Google Cloud Run (as indicated by the .run.app URLs in the environment context). The static assets generated by npm run build can be served by a lightweight Nginx container or an Express static server.
9. Future Enhancements
Retrieval-Augmented Generation (RAG): Implementing a local vector database (e.g., using IndexedDB and client-side embeddings) to allow semantic searching across hundreds of past review documents.
Collaborative Editing: Integrating WebSockets or WebRTC to allow multiple reviewers to co-author the Markdown report simultaneously.
Automated FDA Database Cross-referencing: Building API integrations to automatically query the FDA MAUDE database or 510(k) releasable database based on extracted device names.
10. 20 Comprehensive Follow-up Questions
To ensure the successful continuation, scaling, and refinement of the SmartMed Review 3.0 platform, the following technical and product-oriented questions must be addressed by the engineering and stakeholder teams:
PDF Parsing Limitations: How does the current pdfjs-dist implementation handle scanned documents (non-searchable PDFs) or documents with complex embedded tables and images? Should we integrate an OCR fallback mechanism?
Context Window Overflow: When a medical device submission exceeds the maximum token limit of gemini-3.1-pro-preview, what exact chunking and overlap strategy is implemented in DocIngestion.tsx to prevent loss of critical context?
State Persistence Security: If localStorage is used in store.ts for persisting user sessions, how are we ensuring that no Protected Health Information (PHI) or proprietary trade secrets from the submissions are accidentally cached to the disk?
Mermaid Diagram Security: Since mermaid renders diagrams based on markdown text, what sanitization steps are in place within MarkdownEditor.tsx to prevent Cross-Site Scripting (XSS) attacks if a malicious payload is injected into the markdown?
AI Hallucination Mitigation: In the GuidanceGenerator.tsx, what specific prompting techniques or post-generation validation steps are used to ensure the AI does not hallucinate regulatory clauses or FDA guidance documents?
Telemetry Data Handling: Does the TelemetryTerminal.tsx transmit any data back to a central server, or is it strictly a local diagnostic tool? If transmitted, how is it anonymized?
Theme Extensibility: The index.css file hardcodes 10 Pantone themes. How difficult would it be to allow enterprise users to define custom CSS variables for white-labeling the application?
Zustand Store Scalability: As the application grows to include SubmissionTriage and ReviewReport, will the single store.ts file become a bottleneck? Should we adopt a slice-based pattern for the Zustand store?
API Rate Limiting: How does the application gracefully handle HTTP 429 (Too Many Requests) errors from the Gemini API? Is there an exponential backoff and retry queue implemented?
Offline Capabilities: Is there any requirement for this application to function offline or in air-gapped environments, and if so, how does that impact the reliance on the cloud-based Gemini API?
Component Reusability: Are the Radix UI components wrapped in reusable custom components within src/components/ui/, and do they strictly adhere to the class-variance-authority (cva) patterns for consistent styling?
Markdown Export Formats: Currently, MarkdownEditor.tsx edits markdown. What libraries or services will be used to convert this markdown into FDA-compliant PDF or DOCX formats for final submission?
Testing Strategy: What is the automated testing strategy (Unit, Integration, E2E) for the AI-driven components, given that LLM outputs are non-deterministic?
User Authentication: The current architecture does not explicitly detail an authentication flow. How will users authenticate, and how will Role-Based Access Control (RBAC) be implemented for different reviewer levels?
Prompt Versioning: As regulatory guidelines change, prompts will need updating. How are the system prompts versioned and managed outside of the core application code?
Streaming Interruption: If a user navigates away from the GuidanceGenerator tab while a Gemini stream is actively generating, how is the stream aborted to save bandwidth and compute resources?
Accessibility Audit: Have the custom Radix UI implementations and the complex Markdown Editor been audited for screen reader compatibility and keyboard navigation compliance (WCAG 2.1 AA)?
Dependency Management: The project relies on specific versions of react-pdf and pdfjs-dist which often have strict worker file requirements. How is the PDF worker script being served in the Vite build process?
Triage Logic Implementation: For the SubmissionTriage placeholder, will the logic be purely AI-driven, or will it utilize a hybrid approach combining deterministic rules (e.g., checking for the presence of required sections) with AI analysis?
Audit Trails: In a highly regulated environment like medical device review, 21 CFR Part 11 compliance is often required. How will the system track and log who generated what guidance, and what manual edits were made to the AI-generated reports?
