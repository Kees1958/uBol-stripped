uBol-stripped — Smart ad & tracker blocker
uBol-stripped blocks ads, trackers and privacy threats while you browse — without slowing your browser down. It works silently in the background the moment you install it. No account, no settings, no hassle.

Chrome webstore: https://chromewebstore.google.com/detail/ubol-stripped/femdnbckdgobaelpmbbajpidneljkjaa?pli=1

<img width="317" height="384" alt="image" src="https://github.com/user-attachments/assets/a986406f-279b-44eb-b516-f89a5c1be396" />


HOW UBOL-STRIPPED COMPARES TO UBO-LITE

uBO Lite ships with more filter lists enabled by default — EasyList, EasyPrivacy, Peter Lowe and others — which gives broader coverage but can block cookie consent flows that prevent login on some sites. uBol-stripped uses only three well curated set (Kees1958, AdGuard Base, AdGuard tracking parameters) and only precesses rules from extended EU-zone and 5 Eyes Countries (and it only offers to add additional EU-langauge filters). That is why it is called ¨stripped". On the plus site uBol-stripped has gained some AG-skills, it processes all scriptlets present in the AG base filter. 

<img width="780" height="503" alt="image" src="https://github.com/user-attachments/assets/c507f578-1a8b-423e-b05e-1bd9a4119d37" />


* Reason why AdGuard raw entries generate more DNR rules is that they are finer grained (with lower risk of website breakage and DNR rules are handled super efficient by the browser).
* Reason why uBol-stripped does not offer generic cosmetic filters (and only processes native procedural filters) is to compensate for the additional AdGuard scriplets
   (so page injection size is more or less the same as uBO-lite in default optimal mode).



FOR USERS WHO WANT MORE CONTROL (OR LIKE THE HIGH-TECH OF UBO-LITE & THE HIGH TOUCH OF AG-MV3)

1. Create custom cosmetic rules (uBO-lite element picker)

Draw a box around any annoying element on a page to permanently hide it
<img width="1197" height="582" alt="image" src="https://github.com/user-attachments/assets/369ed757-c5d4-4beb-8f95-5adc786c937a" />
_


2. Create custom DNR rules
Watch live which external services a website contacts and block them with one click
<img width="1488" height="650" alt="image" src="https://github.com/user-attachments/assets/d6a6585c-4390-44e1-a63b-c0f2bf28c55a" />
_


3. Import ABP-rules in one place
Paste your own ABP/uBlock filter rules
<img width="884" height="706" alt="image" src="https://github.com/user-attachments/assets/860c189f-4402-48e7-9a24-afa2df3f97d6" />
_


4. Enable advanced Security & Privacy options
<img width="861" height="809" alt="image" src="https://github.com/user-attachments/assets/6f39e4ff-cc3e-4c10-87a6-1b5ac753598f" />
_


5 Set per-site blocking levels in a simple text editor (This is the existing Filtering mode panel which is hidden behind developer mode)
<img width="875" height="275" alt="image" src="https://github.com/user-attachments/assets/7bc7b9f2-40cc-492b-8ba6-75c4803173f0" />
_


PRIVACY FIRST
uBol-stripped contains no analytics, no telemetry and no ads. It never sends any data about your browsing to anyone. Filter lists are compiled and bundled inside the extension — no external servers are contacted except to check one optional list of suspicious hosting domains when you enable that feature.


WHAT IS CHANGED?

In layman's terms we (Claude AI and me) used the strong bones of uBO-lite and stripped it (removed less well programmed features to keep up with AdGuard Mv3). Next we added some open source AdGuard skills (it is open source so why not use what is available in stead of competing as a one man band with a team of developers). Next I added some features which in my opinion were missing in uBO-lite (see pictures). To make the code easy to understand, extend and debug I refactored the strong uBO-lite bones using four (old structured programming) principles (nerd alert): 

1. SEPARATION OF CONCERNS — HTML, CSS AND JAVASCRIPT
Every file has exactly one job. HTML describes the structure of the page. CSS handles all visual styling. JavaScript handles all behaviour. No styling is written inside JavaScript. No logic is embedded in HTML. This means you can change how something looks without touching the code that makes it work, and vice versa. Each dashboard tab, each popup panel and each background module is its own isolated file.

2. FIVE-TIER FOLDER STRUCTURE
The codebase is organised into five layers, and code is only allowed to call downward — never upward or sideways. This means a bug in the UI can never corrupt the filter logic, and the filter logic can never accidentally write to the UI. Every dependency is explicit and traceable (ui — what the user sees and clicks, workflow — the service worker that coordinates everything, core — the actual blocking and filtering logic, util — small shared helper functions, data — storage, configuration and external API calls)

3. STATE VECTORS AND SWIMMING LANES
Every multi-step process owns a single named state object — called a state vector — that holds all its data and a phase field describing exactly where in its lifecycle it currently is. For example the monitor session can only be in one of four states: idle, starting, running or paused. A guard function checks the phase at the start of every operation and logs a warning if something tries to happen out of order. This means bugs that used to be silent — a session that got stuck halfway, a timer that never fired — are now immediately visible in the browser console with the exact step that failed.
The service worker startup sequence follows the same pattern: it tracks whether it is booting, starting, ready or in error, and records which sub-step it was on if something goes wrong. If the extension ever fails to start, the error message tells you exactly where.

4. MAINTAINABILITY AND BOUNDARY GUARDS
Every capacity limit — the maximum number of custom DNR rules, the maximum number of cosmetic rules, the ID ranges for each rule type — is defined in a single file (dnr-budgets.js) and imported everywhere else. Changing a limit means editing one number in one place. Every module that manages a resource also owns the cleanup of that resource. Timers live inside the state vector they belong to, not scattered across the module. Every future scaling limitation is marked with a comment explaining what the ceiling is and what a future developer would need to change to raise it.

