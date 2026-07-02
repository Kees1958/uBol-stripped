Privacy Policy for uBol-stripped
Last Updated: July 2026

Introduction
uBol-stripped is committed to protecting your privacy. This Privacy Policy explains what information the extension processes and how it is used.

Information We Collect
uBol-stripped does not collect, store, transmit, or share any personal information about users.

Specifically, uBol-stripped does not:


Collect personally identifiable information (PII)
Track user activity or browsing behaviour
Store or transmit browsing history
Collect or read the content of any web page
Create user profiles or analytics data
Contact any external server or API


How uBol-stripped Works
The extension registers filtering rules with Chrome's native Declarative Net Request (DNR) engine. These rules block or allow third-party network requests based on the filter rules you have written yourself. The filtering happens entirely inside the browser — the extension never reads, inspects, or transmits the content of any request.

Cosmetic filters (rules that hide page elements or run a scriptlet you authored) are applied locally through Chrome's User Scripts API, using only the filter text you entered in the extension's own filter editor. No filter content, page content, or browsing data ever leaves your device.

When you toggle filtering on or off for a site from the extension's popup, the extension updates the set of active DNR rules for that domain. Nothing is sent anywhere.

Third-Party Services
uBol-stripped does not use any third-party services. It does not connect to any external API, analytics platform, or remote server of any kind. All processing happens locally within your browser using Chrome's built-in extension APIs.

Data Storage
The following data is stored locally on your device using chrome.storage.local:


Your own custom filter rules (network and cosmetic), written by you in the filter editor
Your per-site filtering state (on/off)
Basic extension settings (e.g. whether tabs are auto-reloaded after a change)


None of this data is ever transmitted outside your browser.

Important Notice
uBol-stripped does not upload, transmit, or share:


The content of any web page you visit
Any request payload or response data
Any personal data of any kind
Your browsing history
Any information about which websites you visit


Changes to This Privacy Policy
This Privacy Policy may be updated from time to time. Any changes will be reflected by updating the "Last Updated" date at the top of this document.
