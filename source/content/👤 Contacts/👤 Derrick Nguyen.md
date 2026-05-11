---
aliases:
contact_type:
date-met:
location-met: University of Missouri - Columbia
last-contact: 2026-05-04
based: Columbia, Missouri
hometown: Springdale, Arkansas
birthday: 05-04
type: contact
---
# Alicia Garza

>[! ]
>**Linked In** 
>
### Appearances

```dataviewjs
// Name of this contact note, e.g. "👤 Derrick Nguyen"
const contactName = dv.current().file.name;
const contactTag = "[[" + contactName + "]]";

// All daily notes with logs
const pages = dv.pages('"Map/Daily"').where(p => p.log);

let structured = [];
let legacy = [];

// Helper to render other contacts on separate lines using <br> for tight spacing
function renderContacts(contactsText, selfName) {
    if (!contactsText) return "";
    return contactsText
        .split(/,\s*/)
        .filter(c => !c.includes(`[[${selfName}]]`))
        .map(c => c.trim())
        .join("<br>"); // HTML line break for single spacing
}

for (let p of pages) {
    let logs = p.log;
    if (!Array.isArray(logs)) logs = [logs];

    for (let raw of logs) {
        if (!raw) continue;
        const text = String(raw);

        if (!text.includes(contactTag)) continue;

        const parts = text
            .split("|")
            .map(s => s.trim())
            .filter(Boolean);

        const isStructured =
            parts.length > 1 &&
            parts.slice(1).some(seg => /^why:/i.test(seg) || /^remember:/i.test(seg));

        if (isStructured) {
            const contactsPart = parts[0] ?? "";
            let why = "";
            let remember = "";

            for (const seg of parts.slice(1)) {
                const lower = seg.toLowerCase();
                if (lower.startsWith("why:")) why = seg.slice(4).trim();
                else if (lower.startsWith("remember:")) remember = seg.slice("remember:".length).trim();
            }

            structured.push({
                source: p.file.link,
                contacts: contactsPart,
                why,
                remember
            });
        } else {
            legacy.push({ source: p.file.link, details: text });
        }
    }
}

// NEW-STYLE LOGS
if (structured.length) {
    dv.header(4, "Recent structured logs");
    dv.table(
        ["Source", "Contacts", "Why this convo?", "Remember for future?"],
        structured
            .sort((a, b) => b.source.path.localeCompare(a.source.path))
            .map(r => [
                r.source,
                renderContacts(r.contacts, contactName),
                r.why,
                r.remember
            ])
    );
}

// OLD LOGS
if (legacy.length) {
    dv.header(4, "Legacy logs (free text)");
    dv.table(
        ["Source", "Details"],
        legacy
            .sort((a, b) => b.source.path.localeCompare(a.source.path))
            .map(r => [r.source, r.details])
    );
}
```
