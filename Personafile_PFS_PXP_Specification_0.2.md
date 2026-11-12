# Personafile Format (PFS) 1.0 and PXP Discovery Profile

**Status:** Proposed specification — Editor's Draft 0.2 (12 November 2026)  
**Maturity:** Experimental; not an adopted Internet standard  
**Scope:** Local-first persona documents, optional embedded elements, discovery, rendering, and API/form integration.

> **Design goal:** An ordinary UTF-8 text file, with any filename or extension, can define a conversational persona. A PXP-capable browser recognizes the file by a deterministic header and interprets optional inline elements. The same persona may be opened locally or from a website. No new URI scheme or new network transport is required.

## 1. Terminology and design boundaries

- **Personafile:** A portable text document containing a signature, metadata and free-form body.
- **PFS:** The Personafile Format Specification, including syntax and meaning of `pf:` elements.
- **PXP:** The persona discovery/exchange behavior implemented by a client, initially using existing `file:` and `https:` resources. PXP in this draft **does not define a new transport protocol**.
- **PXP browser:** Any client capable of parsing and presenting Personafiles using an available conversational runtime. It may run on desktop, web, mobile, or CLI.
- **Persona:** The primary conversational identity described by the document.
- **Resource:** A local file, directory, HTTPS resource, or other explicitly supported URI target.
- **Frontal persona:** An optional default persona for a local folder or website, like an entry page.
- **Embedded content:** A referenced Personafile whose body is incorporated into the primary persona's context. Embedding **does not** create a separate running agent.

### 1.1 Normative wording

**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** indicate implementation requirements in this proposed specification. They do not imply official standards status.

### 1.2 Core principles

1. Plain text remains primary: no XML root, Markdown, JSON, scripts, or `pf:` elements are mandatory.
2. File identification is **content-based**, not extension-based.
3. Existing URI rules and HTTPS are reused; no `pxp://` is required.
4. Embedded elements are **declarative**. They do not grant privileges.
5. APIs and forms are independent and optional.
6. Rendering is independent of the host platform.
7. A static document needs a separately supplied AI runtime for live conversation.

## 2. File identification and envelope

### 2.1 Byte-level identification

A conforming file MUST begin at byte zero with the exact ASCII bytes:

```text
PERSONAFILE/1.0
```

Immediately after this sequence MUST be LF (`0A`) or CRLF (`0D 0A`). No UTF-8 byte-order mark, whitespace, or comments may precede the signature. The document MUST be UTF-8. The file MAY have any extension or none. The filename alone MUST NOT establish that a file is a Personafile.

### 2.2 Header layout

The second line MUST consist of exactly three hyphens: `---`. Each subsequent metadata line MUST use `Key: Value` syntax. A line consisting of exactly `---` terminates metadata and begins the body on the following line.

```text
PERSONAFILE/1.0
---
Persona-ID: chef-emma
Name: Chef Emma
Version: 1.2.0
---
I am Chef Emma, a cooking assistant.
```

The body may be empty. It is **free-form text**; a `.md` file may contain Markdown but Markdown processing is not required by PFS.

### 2.3 Metadata grammar

- Metadata keys are ASCII letters followed by zero or more ASCII letters, digits, or hyphens: `[A-Za-z][A-Za-z0-9-]*`.
- Metadata key matching is ASCII case-insensitive. Canonical display spelling is recommended.
- A key/value separator is the **first colon** on the line; later colons belong to the value.
- Values are single-line UTF-8 text; spaces and tabs immediately after the colon and at the end are ignored.
- Duplicate keys (under case-insensitive comparison) are invalid.
- Unknown valid keys MUST be preserved by editors and MAY be ignored by readers; they have no executable effect.
- Empty lines, multiline/folded values, and comments **inside metadata** are not supported in this version.
- The header MUST be no larger than 64 KiB in UTF-8 bytes, including the signature and delimiter lines.

### 2.4 Core fields

| Field | Requirement | Meaning |
|---|---|---|
| `Persona-ID` | MUST | Stable identifier *within its containing resource scope*, e.g. `chef-emma`; not proof of global identity |
| `Name` | MUST | Display name, nonempty |
| `Description` | MAY | Human-readable summary |
| `Version` | MAY | Revision of the persona, independent of `PERSONAFILE/1.0` |
| `Language` | MAY | Preferred language tag, e.g. `en` or `fr` |
| `Body-Type` | MAY | Advisory body presentation type; defaults to `text/plain`; `text/markdown` may be used |
| `Publisher` | MAY | Claimed publisher URI; **not** verified identity |
| `License` | MAY | Stated reuse terms |

The `Persona-ID` MUST be nonempty and SHOULD use `[A-Za-z0-9][A-Za-z0-9._-]*`. In version 1.0 it is scoped to its file or container, not globally resolvable on its own. Neither `Endpoint` nor network access is required.

### 2.5 Minimal valid document

```text
PERSONAFILE/1.0
---
Persona-ID: hello
Name: Hello
---
You are a friendly assistant.
```

## 3. Body syntax: optional `pf:` elements

The body is ordinary text. A parser MAY recognize optional XML-inspired inline tags with the prefix `pf:`. PFS does **not** require the body to be a well-formed XML document and does **not** adopt the full XML specification. Tags can appear amid prose, in text files of any extension.

### 3.1 Element grammar (PFS subset)

- Element names are lower-case ASCII and case-sensitive: `pf:a`, `pf:embed`, etc.
- Start tags use `<pf:name ...>`; empty tags use `<pf:name ... />`; matching closing tags use `</pf:name>`.
- Attribute names are lower-case ASCII with internal hyphens; values MUST be double-quoted strings.
- Literal `&`, `"`, `<`, `>` inside attribute values MUST be represented by `&amp;`, `&quot;`, `&lt;`, `&gt;` respectively. These four entities are decoded once; no other entity syntax is recognized.
- A tag is recognized only if it fully satisfies this grammar and has a registered element name.
- Unsupported or malformed markup MUST be left as plain body text and MUST NOT trigger effects.
- A recognized element can be surrounded by ordinary prose. Outside known paired tags, regular body text has no mandated markup semantics.
- PFS-aware editors SHOULD preserve unknown text and unknown XML-like tags verbatim.
- Only recognized PFS elements are interpreted; no JavaScript or arbitrary XML evaluation occurs.
- To show literal PFS markup, write `&lt;pf:a ...&gt;` in body text; a plain-text renderer MAY display these escapes literally. If a client implements Markdown rendering, it MUST NOT activate tags within fenced or inline code spans.

For initial interoperability, parsers SHOULD support recognized paired elements without nested PFS elements inside them, except the `pf:form` children defined in §6. Non-nested inline links are sufficient for version 1.0. A future grammar revision may standardize richer nesting.

### 3.2 Element registry (core)

| Element | Required attributes | Meaning |
|---|---|---|
| `<pf:a>` | `pref` | Navigable reference to a resource |
| `<pf:embed />` | `pref` | Incorporate a referenced Personafile's body into the **main** persona |
| `<pf:asset />` | `src` | Declare a media resource, such as image or voice |
| `<pf:section>` | `id` | Label a semantic text section and anchor target |
| `<pf:anchor />` | `id` | Create a named target without a section |
| `<pf:api />` | `src` | Declare an OpenAPI description source; no automatic execution |
| `<pf:form>` | none | Declare a platform-neutral data collection/submission form |

Each element is OPTIONAL. Elements omitted from this list, such as `pf:script`, `pf:tool`, and `pf:import`, have **no normative semantics in 1.0**. The profile intentionally avoids inventing a scripting language.

### 3.3 Links and anchors

```text
See <pf:a pref="./sales.txt">Sales</pf:a>.
See <pf:a pref="https://example.com/help">Help center</pf:a>.
Jump to <pf:a pref="#pricing">pricing</pf:a>.
<pf:anchor id="pricing" />
```

- `pref` means **persona/resource reference**. Its value is a standard URI reference, not a new URI scheme.
- An `pf:a` MAY point to a non-Personafile resource; in that case the client SHOULD use its normal resource handler.
- The reference MAY be relative or absolute.
- `pf:section` uses `<pf:section id="pricing">...content...</pf:section>`; its content belongs to the main document.
- Anchor IDs MUST be unique within the resolved document and SHOULD match `[A-Za-z][A-Za-z0-9._-]*`. Unresolved fragments are nonfatal navigation errors.
- Links do not execute, embed, or elevate the destination.

### 3.4 Embedded Personafiles

```text
I am Alex, a travel assistant.
<pf:embed pref="./paris-knowledge.txt" />
I help visitors plan trips.
```

`pf:embed` MUST resolve to a valid Personafile. The browser parses it, **incorporates the embedded document's body as part of the single primary persona**, and preserves the origin boundary internally. Its metadata remains source attribution; the main document's `Persona-ID` and `Name` remain authoritative for the active persona. The browser MUST NOT create an independent agent or session solely because of `pf:embed`.

To avoid ambiguities, the initial version MUST treat embedded content as **supporting context**, not as a way to override system rules or gain tools/permissions. Any instructions in embedded content remain untrusted subordinate data. Embedded `pf:api` and `pf:form` declarations MUST NOT automatically become active; the main Personafile must declare them directly if needed. Embedded text MAY contain links and assets resolved relative to the embedded document's own location.

Recursive inclusion MUST detect cycles using canonical resolved document URIs and abort the repeated inclusion. Recommended hard limits: 5 levels of nesting and 2 MiB of total expanded text by default; implementations MAY enforce stricter limits and MUST indicate partial loading when truncating. Remote embeds require explicit network permission under browser policy.

### 3.5 Assets

```text
<pf:asset src="./images/avatar.png" type="image/png" />
```

`src` is a URI reference. `type` is an optional media type hint; content MUST be checked before use. The browser MAY render a preview and MUST NOT execute asset content merely because the document refers to it.

## 4. URI resolution and resource discovery

PFS uses existing URI references. Relative references MUST be resolved against the containing document's actual base URI using RFC 3986 §5. A Personafile loaded from an embedded source retains its **own base URI** for references inside that embedded source.

### 4.1 Exact-target resolution

When following a `pref`, a client MUST first resolve the reference to an exact target:

1. `#fragment`: navigate within the current document; do not fetch a different resource.
2. A local **file**: read bytes and inspect the content signature. If not a Personafile, offer normal file handling.
3. A local **directory**: look for a default entry file named exactly `personafile` in that directory. Other Personafiles MAY be listed after permissioned scanning.
4. An HTTP(S) **resource URL**: request the exact resource, with `Accept: application/personafile, text/plain;q=0.9, */*;q=0.1` as a suggested unregistered media-type preference. If the returned bytes have a valid PFS envelope, open as a persona; otherwise use a fallback content handler.

A website URL is not automatically assumed to have an associated persona just because it returns HTML. Discovery is an additional step, described next.

### 4.2 Website-level discovery (proposed profile)

For an HTTPS **origin**, the PXP browser MAY try:

```text
https://example.com/.well-known/personafile
```

This is a **proposed** well-known suffix, not an IANA-registered one. Registration would be needed before claiming standards-compliant globally interoperable use. The browser MUST not blindly transform a deep path like `/support/article` to the origin's home persona without user intent or a documented site mapping.

A PXP-enabled site MAY also serve different representations at `/` according to the HTTP `Accept` header, returning HTML for ordinary web clients and PFS for clients requesting the proposed PFS media type. Both behaviors are optional and do not require a new transport.

When HTTP retrieval is used, HTTPS SHOULD be required by default except for explicitly authorized local development. The server SHOULD use `Content-Type: text/plain; charset=utf-8` until a dedicated Personafile media type is registered. The signature remains authoritative for PFS file detection.

### 4.3 Folder-level discovery and frontal persona

A local directory MAY contain numerous independent Personafiles, any filename or extension. The optional file named exactly `personafile` serves as its **frontal persona**, analogous to a conventional index entry. A browser opening the folder SHOULD load this default when present, otherwise display a permissioned directory listing. It MUST NOT silently read all nested folders or unrelated user files merely because the folder contains a Personafile.

Example:

```text
MyCompany/
  personafile       (front desk)
  sales.txt         (independent persona)
  support.md        (independent persona)
  accounting        (independent persona, no extension)
  notes.txt         (ordinary file)
```

The frontend can link to the others with `<pf:a pref="./sales.txt">Sales</pf:a>`. Links open separate personas; they do not embed them. A directory scan MUST validate content signatures; `.md` and `.txt` extensions alone do not qualify a file as a persona.

## 5. API capability declarations

```text
<pf:api src="https://api.example.com/openapi.json" />
```

This optional declaration refers to an **OpenAPI** description, which MAY expose many operations. PFS MUST NOT duplicate all endpoint schemas in the Personafile. The PXP browser MAY retrieve and parse supported OpenAPI versions, discover operations, and expose only explicitly authorized operations to the conversational runtime. `pf:api` does not execute API calls and does not grant network permission or credentials.

Optional `id` provides a local handle. Optional `tags` provides a comma-separated **presentation filter** matching OpenAPI operation tags; this is not an authorization filter. For example:

```text
<pf:api id="shop" src="https://shop.example.com/openapi.json" tags="catalog,orders" />
```

An implementation MUST reject unsupported OpenAPI syntax or operations cleanly. Authentication is browser-managed and MUST NOT be stored as secrets in Personafile. An API descriptor MUST be considered untrusted input. The browser MUST enforce user or administrator authorization, least privilege, request validation, rate limits as appropriate, and confirmation for consequential actions (such as orders, payments, deletions or sending messages).

## 6. Platform-neutral forms

Forms are independent of APIs: a form MAY submit directly to a URL, or MAY collect values locally without calling a service. A form is a **declarative specification of fields and an optional action**, not an HTML document.

### 6.1 Form elements

```text
<pf:form id="contact" title="Contact support" action="https://example.com/contact" method="POST">
  <pf:input name="email" type="email" required="true" />
  <pf:textarea name="message" required="true" />
  <pf:submit label="Send" />
</pf:form>
```

Core child elements:

- `pf:input`: `name` REQUIRED; `type` defaults to `text` and may be `text`, `email`, `number`, `search`, or `checkbox`; `label` OPTIONAL; `required` OPTIONAL boolean (`true` or `false`).
- `pf:textarea`: `name` REQUIRED; `label` OPTIONAL; `required` OPTIONAL.
- `pf:submit`: `label` OPTIONAL, default `Submit`. Maximum one `pf:submit` per form in 1.0.

`pf:form` attributes: `id` OPTIONAL, `title` OPTIONAL, `action` OPTIONAL, `method` OPTIONAL. If `action` is omitted, the form submits **locally to the active persona/session**, with no network call. If `action` is present, `method` defaults to `GET`; version 1.0 supports only `GET` and `POST` for remote form submission. `method` without `action` is invalid.

For remote forms, encoding defaults to `application/x-www-form-urlencoded; charset=UTF-8`: `GET` appends fields as query parameters, and `POST` sends fields in the request body. Fields are submitted in document order. Duplicate field names are invalid for simplicity, and a required field must pass its declared type validation. Unknown input types are invalid. Additional request body types (JSON, multipart) are **not** specified in this version.

### 6.2 Renderers

The browser MAY render identical form semantics as HTML controls, Android native UI, iOS UI, CLI prompts, voice questions, or conversational interactions. UI details are implementation choices. It MUST preserve field names, data types, required constraints, selected action and method, and confirmation/validation rules across renderers.

- **Web:** render as native HTML controls or equivalent.
- **Android:** render with native views, XML-defined layouts, or Compose.
- **CLI:** prompt for each field, or prepare an equivalent preview/`curl` command for an approved HTTP submission.
- **Conversational:** the model may help collect field values, but submission remains browser-controlled.

A local form result is delivered as structured session data to the primary persona. Submission MUST NOT imply permanent storage or onward transmission. External form submissions MUST display their destination and request confirmation; the browser MUST avoid leaking private user data to undeclared targets.

### 6.3 Forms and APIs can coexist without coupling

```text
<pf:api src="./openapi.json" />
<pf:form id="preferences" title="Preferences">
  <pf:input name="city" type="text" />
  <pf:submit label="Continue" />
</pf:form>
```

Here the API describes optional capabilities; the form collects preferences for the active persona. There is no implicit API call.

## 7. Processing algorithm

A PXP browser SHOULD implement the following steps:

1. **Resolve** the user-selected file, folder, or HTTPS URI under normal access controls.
2. **Discover** the exact Personafile or approved default entry resource.
3. **Detect** the required byte signature; reject invalid envelopes.
4. **Parse** metadata, validating required fields and duplicates.
5. **Tokenize** recognized optional `pf:` elements in the free-form body; preserve ordinary text.
6. **Resolve** linked and embedded resources relative to each source's own base URI.
7. **Incorporate** valid embedded content into the *main* persona context with source boundaries; detect cycles and enforce limits.
8. **Render** available links, forms, and assets in the host platform.
9. **Offer conversation** if the browser has access to an AI runtime; otherwise it MAY provide document viewing and navigation only.
10. **Execute** API operations or form submissions only with appropriate authorization, validation and confirmation.

No processing step allows document instructions to override browser security policy, user approvals or system-level permissions.

## 8. Security and trust model

Personafiles are **untrusted content** by default, whether local or remote. A valid signature proves format only, not authorship. `Publisher` is a claim, not verification. Browser implementations MUST NOT treat ordinary persona prose or embedded content as privileged system instructions. They SHOULD establish a separate trusted instruction layer that controls tools, data access, and execution.

Key protections:

- Network calls and local file reads require permission scoped to the resource.
- No arbitrary local filesystem traversal from `pref` without authorization.
- No execution of inline JavaScript, shell scripts, or code from `pf:asset` or embedded content.
- Prevent recursive embed cycles, excessive file sizes, and URL redirect loops.
- Do not forward secrets or private form values to third parties without user authorization.
- Reject unsafe redirect targets and private-network/API SSRF attempts according to browser policy.
- Do not assume APIs or forms are safe merely because they come from the same document.
- Keep source provenance when composing embedded documents; do not silently elevate embedded instructions.
- If an API or asset fails, provide a recoverable error rather than silently misrepresenting successful execution.

Full digital signatures, verified persona identities, remote sessions, event streams, and interactive scripts are **outside the scope** of this draft.

## 9. Complete examples

### 9.1 Smallest local persona (any filename)

```text
PERSONAFILE/1.0
---
Persona-ID: emma
Name: Emma
---
You are Emma. Help people cook simple meals.
```

### 9.2 Local company folder with frontal persona

```text
MyCompany/
├── personafile
├── sales.txt
├── support
└── reference/
    └── faq.txt
```

**`MyCompany/personafile`**

```text
PERSONAFILE/1.0
---
Persona-ID: company-home
Name: MyCompany Front Desk
Description: Main entry point for visitors
---
Welcome to MyCompany. Tell me what you need.
For purchases, visit <pf:a pref="./sales.txt">Sales</pf:a>.
For assistance, visit <pf:a pref="./support">Support</pf:a>.
```

**`MyCompany/sales.txt`**

```text
PERSONAFILE/1.0
---
Persona-ID: sales
Name: Sales
---
You help people learn about our products.
```

**`MyCompany/support`**

```text
PERSONAFILE/1.0
---
Persona-ID: support
Name: Support
---
You help customers troubleshoot and answer common questions.
```

### 9.3 One persona with embedded knowledge (not an agent)

**`travel.txt`**

```text
PERSONAFILE/1.0
---
Persona-ID: alex
Name: Alex
---
You are Alex, a travel planner.
<pf:embed pref="./paris-knowledge.txt" />
Use the incorporated reference when answering questions about Paris.
```

**`paris-knowledge.txt`**

```text
PERSONAFILE/1.0
---
Persona-ID: paris-facts
Name: Paris Reference
---
Paris has many museums, parks, and neighborhoods.
Check opening hours and prices using current sources before advising.
```

The browser creates **one Alex session** and includes the second document as contextual source material.

### 9.4 Conversational website persona

A site might publish `https://search.example.com/.well-known/personafile`:

```text
PERSONAFILE/1.0
---
Persona-ID: search-home
Name: Search Service
Description: Conversational interface to the website
---
I help visitors search the site's catalog.
<pf:api src="https://search.example.com/openapi.json" />
Users can also search using a standard form:
<pf:form id="search" title="Search" action="https://search.example.com/search" method="GET">
  <pf:input name="q" type="search" required="true" />
  <pf:submit label="Search" />
</pf:form>
```

A browser can show a form or allow conversation. Real results require a functioning API/backend, browser authorization, and an AI runtime for conversational replies. Merely serving a Personafile does not create those services.

## 10. Invalid and non-conforming cases

| Case | Outcome |
|---|---|
| File starts with a UTF-8 BOM | Invalid PFS envelope |
| File has no `PERSONAFILE/1.0` signature | Ordinary file, not PFS |
| Header missing `Name` or `Persona-ID` | Invalid |
| Duplicate header field names | Invalid |
| Badly formed `<pf:api ...>` text | Treat as ordinary text, no API registration |
| Embedded file lacks valid PFS envelope | Embed fails; main persona can still load with warning |
| Embed cycle | Stop repeated inclusion and report warning |
| Form has method `DELETE` in v1.0 | Invalid form definition |
| Form has no `action` | Valid local-session form; no network submission |
| Unreachable API or website | Show unavailable capability; don't invent results |

## 11. Conformance profiles

- **PFS Reader:** MUST identify, validate, and parse the core envelope, preserve ordinary body text, and support the core link and anchor semantics.
- **PFS Composer:** A reader additionally supporting embedded Personafiles with source boundaries and cycle protection.
- **PXP Resource Browser:** A reader supporting permissioned local/HTTP resource resolution, discovery and user navigation.
- **PXP Forms Browser:** A resource browser additionally supporting `pf:form`/fields and consistent submission behavior.
- **PXP API Browser:** A resource browser additionally supporting `pf:api` backed by supported OpenAPI schemas and explicit authorization.
- **Conversational PXP Browser:** Any above profile with a compatible AI runtime. It MUST NOT claim to provide conversation if no such runtime is available.

Clients SHOULD expose supported profiles; no client needs to implement all advanced features to read a basic Personafile.

## 12. Explicitly deferred questions

This is a concrete **draft**, not a claim that every design choice has been ratified. The following need separate review before an interoperable 1.0 release:

1. Registration of an official Personafile media type and `/.well-known/` suffix.
2. Whether to retain `pref` or use conventional `href` in the final syntax.
3. Fully formal grammar for `pf:` markup, quoting/escaping, and Markdown interactions.
4. Verified publisher identity and signed documents.
5. A standard remote session or streaming interface beyond HTTP document retrieval.
6. More form field types, accessibility semantics, defaults, repeated fields and JSON/multipart submissions.
7. OpenAPI version policy, operation allowlists, and credential delegation.
8. Strict interoperability tests and reference parser implementation.

A first working prototype needs only: signature parser, header parser, local folder resolution, `pf:a` navigation, optional `pf:embed` composition, and an AI runtime adapter. Forms and OpenAPI can be implemented independently afterward.

## 13. Standards and reference documents

- RFC 3986, *Uniform Resource Identifier (URI): Generic Syntax* — relative URI resolution: https://www.rfc-editor.org/rfc/rfc3986
- RFC 8615, *Well-Known Uniform Resource Identifiers (URIs)* — well-known registration principles: https://www.rfc-editor.org/rfc/rfc8615
- OpenAPI Specification — API description syntax: https://spec.openapis.org/oas/latest.html
- RFC 9110, *HTTP Semantics* — HTTP request/response semantics and content negotiation: https://www.rfc-editor.org/rfc/rfc9110

---

**Editorial summary:** *Existing URIs locate resources. PXP discovers and exchanges persona representations. Personafile describes the persona. The browser interprets, renders, and safely connects available capabilities.*
