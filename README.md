When I asked in the uBol discussions forum, whether uBol was dropping its design philiosophy by adding features 
(trying to catch up with Adguard Mv3) I was basically told to #$%^-off and write my own extension (so I did and made it better :-)

uBol-stripped is a lean, transparent MV3 content blocker built on Chrome's native Declarative Net Request engine — and a deliberate improvement over uBlock Origin Lite (uBol) for users who want real control over what gets blocked and why. Everything runs inside Chrome's native DNR engine. The extension never reads, inspects, or transmits the content of any web request. No telemetry, no analytics, no external servers of any kind.

<img width="640" height="400" alt="01-popup" src="https://github.com/user-attachments/assets/3419f7cc-e808-4c38-97e4-f2753cbf35af" />


WHY USE uBol-stripped OVER uBo-lite?
More scriptlets, better compatibility. uBol-stripped ships the full AdGuard scriptlet library alongside the uBlock Origin scriptlets. Where uBol fails silently on sites that need AG-specific scriptlets, uBol-stripped handles them correctly.

No fake security. uBol bundles malware and phishing domain lists that update on a 12-hour extension release cycle. By the time a new domain clears that pipeline, it has already done its damage. uBol-stripped replaces that false confidence with two things that actually work: a live-fetched DDNS and free-hosting blocklist pulled directly from GitHub when you enable it, and browser-level resource type blocks (punycode, web bundles, beacons) that stop entire attack categories rather than chasing individual domains.

Point-and-click rule creation — both cosmetic and DNR. uBol has no built-in traffic inspector. uBol-stripped includes a real-time request monitor: open it from the popup, watch every third-party request on the active tab, tick the domains you want blocked, and click Continue. DNR block rules are written instantly. For cosmetic rules, the element picker lets you click any page element to hide it permanently. No filter syntax needed for either.

One place to import rules, not two. uBol splits rule imports across a filters tab and a separate advanced pane. uBol-stripped has a single ABP sandbox editor: paste your network rules, cosmetic rules, and scriptlets in one place, save once, done.

You own the rule budget. uBol-stripped exposes Chrome's full 30,000 dynamic rule capacity with a clear, documented allocation: 1,000 slots for your monitor-created rules, 2 for the DDNS blocklist, and 28,998 for your own imported ABP sandbox rules.
<img width="640" height="400" alt="02-abp-filters" src="https://github.com/user-attachments/assets/1063e5fb-5168-4c66-a753-6f7ea2b6ea9d" />

IMPORT YOUR OWN ABP-FORMAT RULES
The built-in sandbox editor accepts standard ABP-format network rules, CSS cosmetic rules (##), and scriptlets (##+js()), including the full AdGuard scriptlet set. Up to 28,998 compiled DNR rules and 2,000 cosmetic rules.

THREE BUILT-IN FILTER LISTS 
Compiled into the extension and toggled from the ABP-filters tab, just enable them. 

EU & US Ads & Trackers (Kees1958) — 350 rules targeting the most-used ad and tracking networks in Europe and North America
AdGuard URL Tracking Parameter Removal — 897 rules that strip UTM tags, click IDs, and similar tracking parameters from URLs
AdGuard Base Filter — 76,800+ rules covering a broad range of ads, trackers, and malicious content worldwide



SECURITY & PRIVACY TOGGLES
A dedicated Security & Privacy tab — gated behind an advanced-user confirmation — gives fine-grained control over browser-level threats:
Security

<img width="640" height="400" alt="03-security" src="https://github.com/user-attachments/assets/6f13353a-0246-4cf2-b4e5-2b843d64d81c" />


Block punycode links — blocks requests to domains using ACE-encoded (xn--) labels, the encoding attackers use to register look-alike phishing domains. Blocks first- and third-party.
Block suspicious hosting & DDNS — live-fetched list of free hosting platforms and DDNS services frequently abused to distribute malware. Applied as two DNR rules covering 570+ domains and all their subdomains. Block plugin objects — blocks Flash, Java, Silverlight and similar legacy embeds on all sites. Block Web Bundles — blocks the webbundle resource type, exploited to smuggle ad and tracker payloads past conventional network filtering.

Privacy

Block third-party pings — stops hyperlink auditing requests that notify advertisers when you click links. Block beacons (sendBeacon) — blocks navigator.sendBeacon() calls, the mechanism analytics platforms use to fire tracking pings on page unload.


POINT-AND-CLICK RULE CREATION

Create DNR rules — real-time request monitor shows every third-party request on the active tab. Pause, tick domains, continue. Rules are written immediately.
Element picker — click any page element to create a CSS hide rule. Saved per hostname, applied on every future visit.
View and manage — the Custom Rules pane lists all your cosmetic and DNR rules with per-rule deletion and clear-all.

MANAGE THE CUSTOM & COSMETIC RULES YOU CREATED YOURSELF IN ONE PLACE
<img width="640" height="400" alt="05-custom-rules" src="https://github.com/user-attachments/assets/33b8f67c-774b-443d-9a8f-24aaefe6cd07" />




PRIVACY
uBol-stripped does not collect, store, or transmit any personal data, browsing history, or page content. Filter lists are retrieved from GitHub. Full privacy policy included with the
