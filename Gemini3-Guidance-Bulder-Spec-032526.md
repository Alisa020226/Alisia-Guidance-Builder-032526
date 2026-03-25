
Technical Specification: SmartMed Review 3.0
Document Control
Document Version: 3.0.0
Date: March 19, 2026
Status: Final / Approved
Prepared For: SmartMed Regulatory Affairs & Development Team
System Name: SmartMed Review 3.0 (智慧醫材審查指引與清單生成系統)
1. Executive Summary
1.1 Product Vision
SmartMed Review 3.0 is a cutting-edge, AI-powered system designed to revolutionize the Medical Device Regulatory Affairs (RA) review process. By leveraging Google's advanced Gemini Large Language Models (LLMs) and a flexible Agentic architecture, the system automates regulatory guidance processing, intelligent submission triage, and comprehensive review report drafting. Version 3.0 introduces a robust unified document ingestion pipeline (PDF/TXT/MD) with targeted OCR, an agentic skill-execution environment, and a completely overhauled "WOW UI" based on global design standards. The primary objective is to reduce manual review time by 80%, minimize human error, and ensure strict compliance with international medical device regulations.
1.2 Target Audience
Regulatory Affairs (RA) Specialists: Core users who analyze medical device submissions and draft review reports.
Medical Device Reviewers: Officials who evaluate safety and efficacy.
Quality Assurance (QA) Managers: Personnel ensuring submission documents meet regulatory standards.
1.3 Key Objectives (Including V3 Additions)
Universal Document Ingestion (New): Support direct paste or file upload for raw guidance texts (TXT, Markdown, PDF).
Granular PDF OCR & Preview (New): Interactive PDF previewing with page-level selection for text extraction via Python packages or Native LLM Vision models.
Automated Guidance Generation: Transform raw/OCRed regulatory text into structured 2000~3000 word markdown documents containing exactly 3 data tables.
Agentic Skill Execution (New): Allow users to apply custom AI "skills" or prompts directly to the generated guidance documents to fulfill specific downstream tasks.
Intelligent Submission Triage: Automatically categorize submission documents into Required, Not Required, and Optional items.
Comprehensive Report Drafting: Generate 3000-4000 word professional review reports.
Real-time Telemetry & Control: Real-time visibility into the AI's thought process with instant interrupt mechanisms.
WOW UI & Theming (New): A beautifully crafted, highly accessible interface supporting Light/Dark modes, bilingual switching (EN/ZH), and 10 dynamic themes based on official Pantone color palettes.
2. System Architecture
2.1 High-Level Architecture Overview
SmartMed Review 3.0 remains a client-centric application using React 18 and Vite, but introduces a lightweight local Python execution environment (via Pyodide/WebAssembly or an optional microservice) for heavy document processing, maintaining data privacy while vastly expanding capabilities.
code
Mermaid
graph TD
    A[User Interface / WOW UI Pantone Themes] -->|User Input & Config| B(State Management / Context API)
    B --> C{Module Router}
    
    C -->|Tab 1| D[Doc Ingestion & OCR]
    D -->|PDF/Pages| D1{OCR Router}
    D1 -->|Python Packages| D2[Pyodide / Python Microservice]
    D1 -->|LLM Vision OCR| H
    
    C -->|Tab 2| E[Guidance Generator]
    C -->|Tab 3| F[Markdown Editor & Agent]
    C -->|Tab 4| G[Submission Triage]
    C -->|Tab 5| I[Review Report]
    C -->|Tab 6| J[Settings & Theme]
    
    E --> H[Gemini Service Layer]
    F --> H
    G --> H
    I --> H
    
    H -->|REST/gRPC over HTTP| K((Google Gemini API))
    H -->|Streaming Text / JSON| L[Telemetry Terminal]
    L --> A
2.2 Core Architectural Principles
Stateless Processing: Sensitive submission data remains un-persisted. Local PDF rendering and browser-based OCR ensures PII/PHI does not leak.
Component-Driven Design: Granular UI elements.
Asynchronous Streaming: Utilizing for await...of to stream massive context responses smoothly.
Graceful Degradation & Interruption: Deeply integrated AbortController patterns.
3. Technology Stack & Dependencies
3.1 Frontend Framework & Processing
React 18 & Vite: Core application framework.
TypeScript: Enforces strict typing for LLM schemas and UI props.
React-PDF / PDF.js: Renders uploaded PDFs directly in the browser for page selection.
Pyodide / Tesseract.js (Optional microservice): Provides local "Python package" style OCR (e.g., PyMuPDF) directly in the browser or via a lightweight, stateless local Docker container, depending on deployment scale.
3.2 Styling & WOW UI Components
Tailwind CSS: Utility-first styling.
CSS Variables (Pantone Theme Engine): Dynamically injects Pantone color palettes into Tailwind variables.
Lucide React: Iconography.
3.3 Markdown & Visualization
React Markdown & Remark GFM: For rendering complex tables, charts, and lists.
Mermaid JS: Generates the Visual Review Flowchart (WOW 1).
3.4 AI & Machine Learning Integration
Google Gen AI SDK (@google/genai): Used to interface with text and multimodal/vision models (gemini-2.5-flash, gemini-3-flash-preview).
4. Detailed Module Specifications
4.1 Module 1: Document Ingestion & OCR (New)
4.1.1 Purpose
Allow users to easily input premarket submission review guidance via copy-paste, TXT, MD, or complex PDF uploads.
4.1.2 Features & Flow
Upload/Paste Zone: Drag and drop UI.
Interactive PDF Preview: If a PDF is uploaded, react-pdf renders a thumbnail gallery of pages.
Page Selection: User clicks to select/deselect specific pages to process (avoiding blank pages or irrelevant appendices).
OCR Engine Selection: User toggles between:
Python Packages: Uses traditional layout-preserving algorithms (PyMuPDF/Tesseract).
LLM-Based OCR: Sends page images directly to Vision models (gemini-2.5-flash or gemini-3-flash-preview) for high-fidelity text and table extraction.
4.2 Module 2: Guidance Generator (Enhanced)
4.2.1 Purpose
Transform the ingested/OCRed raw text into a highly organized, strict-length Markdown document.
4.2.2 State Variables & Configuration
LLM Model Selection: Dropdown for gemini-2.5-flash or gemini-3-flash-preview.
Target Language: Toggle between English (en) and Traditional Chinese (zh-TW).
Custom Prompt Override: A visible textarea where the user can append custom instructions to the base prompt.
4.2.3 Strict Prompt Engineering Strategy
code
TypeScript
const prompt = `
請根據以下法規內容，生成一份結構化的醫療器材審查指引 (Review Guidance)。
內容長度需嚴格控制在 2000 到 3000 字之間。
請確保整份文件中「剛好包含 3 個 Markdown 表格」來整理關鍵數據或法規對照。
輸出語言：${targetLanguage === 'zh-TW' ? '繁體中文' : 'English'}
用戶自訂附加指令：${userCustomPrompt}

必備內容：
1. 適用範圍 (Scope)
2. 審查重點 (Key Review Points)
3. WOW 3 模擬退件風險評估：常見缺失與風險預測
4. WOW 2 國際法規智能對照：TFDA 規範對照美國 FDA 或歐盟 MDR
5. WOW 1 Mermaid 視覺化審查流程圖：請使用 \`\`\`mermaid 語法包裝
6. WOW 4 法規溯源熱區圖 (Traceability Heatmap) (此部分可作為其中一個表格)

法規內容：
${ingestedText}
`;
4.3 Module 3: Markdown Editor & Agentic Skill Execution (New)
4.3.1 Purpose
Once the 2000-3000 word markdown document is generated, the user enters the Agentic Workspace.
4.3.2 Features
Dual View Editor: Side-by-side raw text/markdown editor and rendered React-Markdown preview.
Export: One-click download as .md or .txt.
Agentic Skill Application (WOW 9):
Users can paste an existing "Skill Description" (a set of specialized system instructions, e.g., "Translate to layman terms," "Extract required testing standards into CSV format").
The Agent executes this skill specifically targeting the generated document in a new chat thread/panel, allowing rapid, specialized document transformations without leaving the SPA.
4.4 Module 4: Submission Triage (Original Logic Maintained)
Categorizes submission documents using strict JSON Schema structured output via the Gemini API, forcing required, not_required, and optional arrays. (See Original Spec for JSON Schema details).
4.5 Module 5: Review Report Generator (Original Logic Maintained)
Drafts 3000-4000 word formal review reports based on submission summaries and guidance. Includes:
WOW 5: Auto-Generated Clarification Q&A.
WOW 6: Tone & Compliance Analyzer.
4.6 Module 6: Telemetry Terminal
A hacker-styled terminal window integrated across all LLM interaction points, displaying streaming network status, payload sizes, token usage, and housing the crucial AbortController kill switch.
4.7 Module 7: Settings & "WOW UI" Theming Engine (New)
4.7.1 Purpose
To provide an aesthetically pleasing, highly personalized workspace that reduces eye strain during long document reviews.
4.7.2 UI/UX Features
Localization: Instant application-wide toggle between English and Traditional Chinese UI text.
Light/Dark Mode: Deep Tailwind integration.
WOW 7 - Pantone Color Palettes: User can select from 10 curated themes. This alters primary buttons, hover states, terminal accents, and Mermaid chart stylings.
Theme 1: Classic Blue (Pantone 19-4052) - Trust & Professionalism
Theme 2: Very Peri (Pantone 17-3938) - Innovation & AI
Theme 3: Emerald (Pantone 17-5641) - Healthcare & Growth
Theme 4: Rose Quartz & Serenity - Calmness & Focus
Theme 5: Illuminating Yellow & Ultimate Gray - Alertness & Foundation
Themes 6-10: Custom refined palettes based on user testing.
5. API Integration Layer
5.1 Dynamic Model Routing
The application dynamically routes API calls based on the module and user selection:
Vision/OCR Tasks: Routed to gemini-2.5-flash or gemini-3-flash-preview using multimodal message parts (Base64 images + Text).
Guidance Transformation: Routed to user-selected model.
Report Generation: Routed to models capable of massive output length and extended reasoning.
6. Summary of "WOW" Features
WOW 1: Mermaid Visual Review Flowchart.
WOW 2: International Regulatory Mapping.
WOW 3: Simulated Rejection Risk.
WOW 4: Regulatory Traceability Heatmap.
WOW 5: Auto-Generated Clarification Q&A.
WOW 6: Tone & Compliance Analyzer.
WOW 7 (NEW): 10 Pantone-based themes with Light/Dark and EN/ZH localization.
WOW 8 (NEW): Granular PDF Preview & Multi-Engine OCR Selection.
WOW 9 (NEW): Agentic Skill Execution Workspace for dynamic document manipulation.
7. Comprehensive Follow-Up Questions
To ensure this specification fully meets all strategic and technical requirements, please consider the following 20 follow-up questions:
Document Ingestion & OCR Processing
For the PDF preview and page selection feature, do you expect users to upload files larger than 100MB, and should we implement client-side chunking/pagination to prevent browser crashes?
When a user selects "Python packages" for OCR, should this run purely in the browser via WebAssembly (Pyodide), or do you prefer a lightweight local Python server (e.g., FastAPI) to handle heavier documents?
If the LLM Vision OCR encounters highly complex, nested tables in a PDF, what is the fallback strategy if the extraction format breaks?
Should the system persist the original uploaded PDF in local browser storage (IndexedDB) so the user doesn't have to re-upload if they refresh the page?
Prompt Engineering & Output Formatting
5. You requested exactly 3 tables in the generated markdown. Are these 3 tables always strictly defined (e.g., Table 1: Scope, Table 2: Traceability, Table 3: Risk), or should the LLM dynamically decide their content based on the text?
6. The target word count is 2000-3000 words. Since LLMs can sometimes struggle with strict word count boundaries, how should the UI handle outputs that fall short of 2000 words? (e.g., auto-prompting it to expand?)
7. When translating regulatory nuances from Traditional Chinese to English (or vice versa), should we append a specific glossary prompt to ensure standard medical device terminology (e.g., using "Premarket Notification" consistently)?
Agentic Skill Execution & Editing
8. For the Agentic Skill feature, will users be inputting simple text prompts, or should we support uploading formal .skill files structured with YAML/Markdown?
9. When an Agent modifies the generated Markdown document, should it overwrite the text directly in the editor, or generate a "Diff/Compare" view so the user can see what the agent changed?
10. Do you want the downloaded .md or .txt files to include metadata headers (e.g., timestamp, model used, original guidance filename)?
WOW UI & Theming
11. For the 10 Pantone color palettes, should these colors also reflect inside the generated Mermaid charts and PDF export styling, or just the application's UI buttons and backgrounds?
12. Should the system remember the user's selected Pantone theme, language, and model preference via localStorage for their next session?
13. Is accessibility (WCAG compliance, high contrast text) a strict requirement when mapping out these specific Pantone colors to the UI elements?
Architecture & API
14. Because gemini-3-flash-preview and Vision OCR can consume significant tokens, do you need an interface within the Settings Tab for users to input their own Google API key, or will this be a centralized enterprise key?
15. Should the Telemetry Terminal capture and display the exact JSON schema validation steps when the Submission Triage module parses the LLM output?
16. How should the application handle rate-limiting (e.g., HTTP 429 errors) from the Gemini API if a user runs multiple OCR pages and guidance generations in rapid succession?
Regulatory & Future Roadmap
17. Are there any specific data-privacy disclaimers or "AI usage warnings" that must be permanently displayed on the UI to comply with internal RA compliance standards?
18. For the Submission Triage module, do you plan to eventually allow users to upload the actual submission documents (e.g., the manufacturer's clinical evaluation report) for the AI to cross-reference against the generated guidance?
19. Would you like a feature to export the entire workspace (the uploaded PDF, the generated Markdown, and the Review Report) as a single compressed .zip package?
20. As we introduce LLM-based OCR, should we add a "Confidence Score" or "Review Needed" highlighter for text where the LLM was unsure about a blurry PDF scan?
