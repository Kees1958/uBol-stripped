uBol-stripped — Smart ad & tracker blocker
uBol-stripped blocks ads, trackers and privacy threats while you browse — without slowing your browser down. It works silently in the background the moment you install it. No account, no settings, no hassle.

<img width="292" height="370" alt="1" src="https://github.com/user-attachments/assets/6f9c9a60-fdde-4636-9f40-32101e98b5b8" />


HOW UBOL-STRIPPED COMPARES TO UBO LITE

uBol-stripped has 3 states on the slider (OFF_BASIC-OPTIMAL), uBO-lite four (also has COMPLETE). 
uBO Lite ships with more filter lists enabled by default — EasyList, EasyPrivacy, Peter Lowe and many others (in my opinion useless malware filters with a slow 12 hour update cycle) — which gives broader coverage but can block cookie consent flows that prevent login on some sites. uBol-stripped uses a smaller, curated set (Kees1958, AdGuard Base, AdGuard tracking parameters) chosen to block ads and trackers with low (website) breakage risk and maintenance (this is also the reason uBol-stripped does not need uBO-unbreak filter).
.



WHAT GETS BLOCKED

Ads and tracking scripts (AdGuard Base + Kees1958 EU/US list)
URL tracking parameters  (AdGuard URL parameter removal list)
Cookie consent banners and popup overlays (in Optimal mode also from AdGuard)
Optional user enabled blocks in the Security & Privacy tab

<img width="884" height="352" alt="image" src="https://github.com/user-attachments/assets/8224452b-0e61-4327-8246-2e145050c9b0" />




FOR USERS WHO WANT MORE CONTROL

1. Create custom cosmetic rules (uBO-lite element picker)

Draw a box around any annoying element on a page to permanently hide it
<img width="1197" height="582" alt="image" src="https://github.com/user-attachments/assets/369ed757-c5d4-4beb-8f95-5adc786c937a" />



2. Create custom DNR rules
Watch live which external services a website contacts and block them with one click
<img width="1488" height="650" alt="image" src="https://github.com/user-attachments/assets/d6a6585c-4390-44e1-a63b-c0f2bf28c55a" />



3. Import ABP-rules in one place
Paste your own ABP/uBlock filter rules
<img width="884" height="706" alt="image" src="https://github.com/user-attachments/assets/860c189f-4402-48e7-9a24-afa2df3f97d6" />



4. Enable advanced Security & Privacy options
<img width="881" height="694" alt="image" src="https://github.com/user-attachments/assets/d9280a2b-adcd-40d2-93f5-9596127baeab" />





5 Set per-site blocking levels in a simple text editor (This is the existing Filtering mode panel)
<img width="875" height="275" alt="image" src="https://github.com/user-attachments/assets/7bc7b9f2-40cc-492b-8ba6-75c4803173f0" />

This (easy to use) panel was hidden behind ¨developer mode" where you can copy paste domains per blocking level.


PRIVACY FIRST
uBol-stripped contains no analytics, no telemetry and no ads. It never sends any data about your browsing to anyone. Filter lists are compiled and bundled inside the extension — no external servers are contacted except to check one optional list of suspicious hosting domains when you enable that feature.

BASED ON UBLOCK ORIGIN LITE
Built on the open-source uBlock Origin Lite (MV3), refactored for simplicity and extended with per-site modes, a live traffic monitor, custom rule management and advanced security & privacy toggles.

WHAT IS CHANGED?
In layman's terms we (Claude AI and me) used the strong bones of uBO-lite and stripped it from (less well programmed features to keep up with AdGuard Mv3). Next we added some open source AdGuard skills (for a one man band it is hard to beat a team of developers). Next I added some features which in my opinion were missing in uBO-lite (see pictures). To make the code easy to understand, extend and debug I refactored the strong uBO-lite bones using four (old structured programming) principles (nerd alert): 

1. SEPARATION OF CONCERNS — HTML, CSS AND JAVASCRIPT
Every file has exactly one job. HTML describes the structure of the page. CSS handles all visual styling. JavaScript handles all behaviour. No styling is written inside JavaScript. No logic is embedded in HTML. This means you can change how something looks without touching the code that makes it work, and vice versa. Each dashboard tab, each popup panel and each background module is its own isolated file.

2. FIVE-TIER FOLDER STRUCTURE
The codebase is organised into five layers, and code is only allowed to call downward — never upward or sideways. This means a bug in the UI can never corrupt the filter logic, and the filter logic can never accidentally write to the UI. Every dependency is explicit and traceable (ui — what the user sees and clicks, workflow — the service worker that coordinates everything, core — the actual blocking and filtering logic, util — small shared helper functions, data — storage, configuration and external API calls)

3. STATE VECTORS AND SWIMMING LANES
Every multi-step process owns a single named state object — called a state vector — that holds all its data and a phase field describing exactly where in its lifecycle it currently is. For example the monitor session can only be in one of four states: idle, starting, running or paused. A guard function checks the phase at the start of every operation and logs a warning if something tries to happen out of order. This means bugs that used to be silent — a session that got stuck halfway, a timer that never fired — are now immediately visible in the browser console with the exact step that failed.
The service worker startup sequence follows the same pattern: it tracks whether it is booting, starting, ready or in error, and records which sub-step it was on if something goes wrong. If the extension ever fails to start, the error message tells you exactly where.

4. MAINTAINABILITY AND BOUNDARY GUARDS
Every capacity limit — the maximum number of custom DNR rules, the maximum number of cosmetic rules, the ID ranges for each rule type — is defined in a single file (dnr-budgets.js) and imported everywhere else. Changing a limit means editing one number in one place. Every module that manages a resource also owns the cleanup of that resource. Timers live inside the state vector they belong to, not scattered across the module. Every future scaling limitation is marked with a comment explaining what the ceiling is and what a future developer would need to change to raise it.

Because I added AdGuard skills (and uBO-lite is more programmed towards efficiency and performance than AdGuard Mv3) I wanted to regain the efficieny and low impact by using old and forgotten programming principles (yes I am 67 and programmed assembler in 16KB mainframes routines) to streamline and optimize the original coriginal code even further. 
