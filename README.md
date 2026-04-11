# NeonFeud

## Overview
Live at https://two-player.app/ NeonFeud is a serverless, Progressive Web App (PWA) trivia/board game engineered for offline-first performance. The application manages complex game states, dynamic audio execution, and data parsing entirely within the client's browser, eliminating the need for a persistent backend connection during gameplay.

## Architecture & Data Flow
The application operates on a decentralized, client-heavy architecture.

* **Frontend Engine (`index.html`, `script.js`):** Vanilla JavaScript handles all game logic, score tabulation, DOM manipulation, and audio rendering. 
* **Data Ingestion (`packs.json`):** Question sets and game configuration data are loaded dynamically from a static JSON file, functioning as a read-only API.
* **Progressive Web App (PWA):** Utilizes a Service Worker (`sw.js`) and application manifest to aggressively cache local assets (audio files, scripts, and JSON data), enabling a native-like installation and persistent offline execution.

## Technology Stack
* **Frontend:** HTML5, CSS3, Vanilla JavaScript (ES6+)
* **Architecture:** Progressive Web App (PWA)
* **Data Storage:** JSON

## Threat Model & Security Considerations
While designed as a standalone entertainment application, the architecture serves as a practical study in client-side trust models and local execution vulnerabilities.

### 1. Client-Side State Manipulation
* **Vector:** All game state variables (scores, current turn, active questions) are stored in client-side memory. An attacker can easily attach a debugger, manipulate variables via the browser console, or alter the DOM to artificially inflate scores or reveal hidden answers.
* **Mitigation Strategy:** In a multiplayer or competitive production environment with a leaderboard, this architecture is inherently flawed. Client-side variables must be treated as untrusted. All score validation and game state progression would need to be moved to an authoritative backend server (e.g., using WebSockets for real-time validation).

### 2. Service Worker Poisoning (PWA Risks)
* **Vector:** The application relies on `sw.js` for caching and offline functionality. If an XSS vulnerability were present—for instance, if user-supplied trivia packs were dynamically loaded without sanitization—an attacker could execute malicious scripts to poison the service worker cache. This would lead to persistent interception of network requests and long-term compromise of the application state.
* **Mitigation Strategy:** Enforcing a strict Content Security Policy (CSP) to restrict the execution of inline scripts and ensuring that any dynamically loaded JSON data is strictly parsed and output-encoded before rendering to the DOM.

### 3. Data Integrity & Insecure Direct Object Reference (Local)
* **Vector:** The game pulls its logic and answers from `packs.json`. Because the client requires full access to this file to function offline, all answers are essentially exposed to the user immediately upon load.
* **Mitigation Strategy:** For competitive integrity, question payloads should be delivered individually per round via a secure API request, rather than exposing the entire dataset to the client at initialization.
