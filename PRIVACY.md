Privacy Policy for uBlock Stripped
Last Updated: August 2026

Introduction
uBlock Stripped is committed to protecting your privacy. This Privacy Policy explains what information the extension processes and how it is used.

Information We Collect
uBlock Stripped does not collect, store, transmit, or share any personal information about users.

Specifically, uBlock Stripped does not:
- Collect personally identifiable information (PII)
- Track user activity or browsing behaviour
- Store or transmit browsing history
- Collect or read the content of any web page
- Create user profiles or analytics data

Filter lists are retrieved from GitHub. No personal data, browsing history, or identifiers are included in these requests.

How uBlock Stripped Works
The extension registers filtering rules with Chrome's native Declarative Net Request (DNR) engine. These rules block or allow third-party network requests based on the filter rules you have written yourself. The filtering happens entirely inside the browser — the extension never reads, inspects, or transmits the content of any request.

Cosmetic filters (rules that hide page elements or run a scriptlet you authored) are applied locally through Chrome's User Scripts API, using only the filter text you entered in the extension's own filter editor. No filter content, page content, or browsing data ever leaves your device.

When you toggle filtering on or off for a site from the extension's popup, the extension updates the set of active DNR rules for that domain. Nothing is sent anywhere.

The "uBO dynamic filtering" panel works the same way: while it's open, it lists the third-party connections the current tab makes and lets you block or allow specific ones, per site. That per-site list is the same kind of local, on-device state as the per-site filtering toggle above — see Data Storage below — and is never transmitted anywhere.

Third-Party Services
uBlock Stripped does not use any third-party analytics, tracking, or advertising services. Filter lists are retrieved from GitHub. All other processing happens locally within your browser using Chrome's built-in extension APIs.

Data Storage
The following data is stored locally on your device using chrome.storage.local:
- Your own custom filter rules (network and cosmetic), written by you in the filter editor
- Your per-site filtering state (on/off)
- Your per-site third-party block/allow overrides ("uBO dynamic filtering" panel)
- Basic extension settings (e.g. whether tabs are auto-reloaded after a change)

None of this data is ever transmitted outside your browser.

Important Notice
uBlock Stripped does not upload, transmit, or share:
- The content of any web page you visit
- Any request payload or response data
- Any personal data of any kind
- Your browsing history
- Any information about which websites you visit

Changes to This Privacy Policy
This Privacy Policy may be updated from time to time. Any changes will be reflected by updating the "Last Updated" date at the top of this document.

Third-Party Data Attribution
This extension bundles a compacted subset of DuckDuckGo's Tracker Radar dataset (used only for the Privacy Inspector's own local, on-device display — never transmitted anywhere) and some cookie-consent-handling logic adapted from DuckDuckGo's autoconsent project. Full attribution and license terms for both are in THIRD_PARTY_LICENSES.md.
