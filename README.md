uBol-stripped — Smart ad & tracker blocker
uBol-stripped blocks ads, trackers and privacy threats while you browse — without slowing your browser down. It works silently in the background the moment you install it. No account, no settings, no hassle.

HOW IT WORKS
Every website gets a three-position slider in the popup. You set it once per site and forget it.

OFF — no filtering on this site (use this if a site is broken)
BASIC — blocks ads and trackers (the default for every site)
OPTIMAL — everything in BASIC, plus automatic removal of cookie consent banners and popup overlays

<img width="294" height="384" alt="image" src="https://github.com/user-attachments/assets/b394c3ea-f81b-4f6f-aed1-675ce1bc6d29" />



WHAT GETS BLOCKED

Ads and tracking scripts (AdGuard Base + Kees1958 EU/US list)
URL tracking parameters  (AdGuard URL parameter removal list)
Cookie consent banners and popup overlays (in Optimal mode also from AdGuard)
Optional user enabled blocks in the Security & Privacy tab

<img width="856" height="746" alt="image" src="https://github.com/user-attachments/assets/6c95cfdf-b25f-41e8-98c9-bbf9e717d4ec" />



FOR USERS WHO WANT MORE CONTROL

1. Create custom cosmetic rules (uBO-lite element picker)

Draw a box around any annoying element on a page to permanently hide it
<img width="1197" height="582" alt="image" src="https://github.com/user-attachments/assets/369ed757-c5d4-4beb-8f95-5adc786c937a" />



2. Create custom DNR rules
Watch live which external services a website contacts and block them with one click
<img width="1488" height="650" alt="image" src="https://github.com/user-attachments/assets/d6a6585c-4390-44e1-a63b-c0f2bf28c55a" />



3. Import ABP-rules in one place
Paste your own ABP/uBlock filter rules
<img width="880" height="705" alt="image" src="https://github.com/user-attachments/assets/654bffba-599f-417b-a010-de7701eb55d3" />


4. Enable advanced Security & Privacy options
<img width="864" height="697" alt="image" src="https://github.com/user-attachments/assets/d96898cc-6592-4db1-979d-702e0d022999" />



5 Set per-site blocking levels in a simple text editor.
This is the existing Filtering mode panel which was hidden behind ¨developer mode"
You can easily copy paste domains per blocking level (OFF- BASIC - OPTIMAL)
<img width="905" height="701" alt="image" src="https://github.com/user-attachments/assets/749a7aff-71a9-4a33-8700-7949bbc5d163" />






PRIVACY FIRST
uBol-stripped contains no analytics, no telemetry and no ads. It never sends any data about your browsing to anyone. Filter lists are compiled and bundled inside the extension — no external servers are contacted except to check one optional list of suspicious hosting domains when you enable that feature.

BASED ON UBLOCK ORIGIN LITE
Built on the open-source uBlock Origin Lite (MV3), refactored for simplicity and extended with per-site modes, a live traffic monitor, custom rule management and advanced security & privacy toggles.

WHAT IS CHANGED?
In layman's terms we used the strong bones of uBO-lite and stripped it from (less well programmed features to keep up with AdGuard Mv3). Next we added some open source AdGuard skills (for a one man band it is hard to beat a team of developers). Next I added some features which in my opinion were missing in uBO-lite (see pictures). To make the code easy to understand, extend and debug we refactored the strong uBO-lite bones using four principles (nerd alert): 

1. SEPARATION OF CONCERNS — HTML, CSS AND JAVASCRIPT
Every file has exactly one job. HTML describes the structure of the page. CSS handles all visual styling. JavaScript handles all behaviour. No styling is written inside JavaScript. No logic is embedded in HTML. This means you can change how something looks without touching the code that makes it work, and vice versa. Each dashboard tab, each popup panel and each background module is its own isolated file.

2. FIVE-TIER FOLDER STRUCTURE
The codebase is organised into five layers, and code is only allowed to call downward — never upward or sideways:

ui/ — what the user sees and clicks
workflow/ — the service worker that coordinates everything
core/ — the actual blocking and filtering logic
util/ — small shared helper functions
data/ — storage, configuration and external API calls

This means a bug in the UI can never corrupt the filter logic, and the filter logic can never accidentally write to the UI. Every dependency is explicit and traceable.

3. STATE VECTORS AND SWIMMING LANES
Every multi-step process owns a single named state object — called a state vector — that holds all its data and a phase field describing exactly where in its lifecycle it currently is. For example the monitor session can only be in one of four states: idle, starting, running or paused. A guard function checks the phase at the start of every operation and logs a warning if something tries to happen out of order. This means bugs that used to be silent — a session that got stuck halfway, a timer that never fired — are now immediately visible in the browser console with the exact step that failed.
The service worker startup sequence follows the same pattern: it tracks whether it is booting, starting, ready or in error, and records which sub-step it was on if something goes wrong. If the extension ever fails to start, the error message tells you exactly where.

4. MAINTAINABILITY AND BOUNDARY GUARDS
Every capacity limit — the maximum number of custom DNR rules, the maximum number of cosmetic rules, the ID ranges for each rule type — is defined in a single file (dnr-budgets.js) and imported everywhere else. Changing a limit means editing one number in one place. Every module that manages a resource also owns the cleanup of that resource. Timers live inside the state vector they belong to, not scattered across the module. Every future scaling limitation is marked with a comment explaining what the ceiling is and what a future developer would need to change to raise it.
