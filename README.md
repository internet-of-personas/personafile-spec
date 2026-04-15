# Personafile Specification

* **Version:** Draft 0.1
* **Status:** Working Draft

The **Personafile** is the public machine-readable descriptor for a digital persona on the **Internet of Personas (IoP)**. It acts as the public handshake document, defining who the persona is, what it can do, and how clients can interact with it over the **Persona Exchange Protocol (PXP)**.

By convention, a persona host should serve this file at:

`https://<authority>/.well-known/personafile.json`

---

## 1. Purpose

The Personafile provides a standard public descriptor that enables clients, registries, and other personas to:

* identify a persona,
* understand its purpose,
* discover its endpoints,
* determine its continuity model,
* inspect its declared capabilities,
* understand its shared interaction views,
* and evaluate basic trust metadata.

The Personafile is intended to be simple, web-native, and interoperable across runtimes.

---

## 2. Design Principles

The Personafile specification is guided by the following principles:

* **Minimality:** the descriptor should remain small and easy to implement.
* **Clarity:** fields should have clear semantics and predictable structure.
* **Interoperability:** different runtimes should be able to consume the same public descriptor.
* **Extensibility:** future versions may add richer trust, discovery, and transport metadata without breaking the core model.
* **Web compatibility:** the descriptor should integrate cleanly with HTTPS, DNS, and `.well-known` conventions.

---

## 3. Publication Convention

A persona should publish its Personafile using the following discovery path:

`https://<authority>/.well-known/personafile.json`

For example:

`https://architect.example.com/.well-known/personafile.json`

This allows standard web infrastructure to resolve persona metadata without introducing a mandatory custom transport layer.

---

## 4. Core Fields

A valid Personafile describes the public interaction surface of a persona.

### 4.1 Required Fields

The following fields are required:

* `spec_version`
* `identity`
* `title`
* `endpoints`
* `state_mode`

### 4.2 Optional Fields

The following fields are optional but recommended:

* `description`
* `owner`
* `version`
* `authentication`
* `views`
* `capabilities`
* `trust_metadata`

---

## 5. Field Definitions

### `spec_version`

The version of the Personafile specification implemented by this document.

Example:

```json
"spec_version": "0.1"
```

### `identity`

The globally unique persona identifier. This should use the `pxp://` scheme.

Example:

```json
"identity": "pxp://architect.example.com"
```

### `title`

A human-readable name for the persona.

Example:

```json
"title": "Reference Builder Persona"
```

### `description`

A short summary of the persona’s purpose.

Example:

```json
"description": "A structural validation persona for architectural plans."
```

### `owner`

The entity, organization, or authority responsible for the persona.

Example:

```json
"owner": "Internet of Personas Project"
```

### `version`

The version of the persona descriptor or published persona profile.

Example:

```json
"version": "0.1.0"
```

### `authentication`

A list of supported authentication methods required or accepted by the persona’s public endpoints.

Example:

```json
"authentication": ["none", "signature"]
```

Possible values may include:

* `none`
* `api_key`
* `oauth2`
* `signature`
* `mtls`

This list is not yet a closed registry in Draft 0.1.

### `endpoints`

An object listing the public endpoints through which the persona accepts PXP interactions.

Supported fields:

* `message`: endpoint for standard request-response exchanges
* `event`: endpoint for event stream delivery or subscription

Example:

```json
"endpoints": {
  "message": "https://architect.example.com/api/v1/pxp/message",
  "event": "wss://architect.example.com/api/v1/pxp/events"
}
```

### `views`

A mapping of shared interaction view names to external schema or contract documents that define the structured state a persona and client may read or write collaboratively.

The Personafile declares the shared data contract for each view. It does not dictate how a client should render that data for humans. Rendering remains the responsibility of the client.

Each value in `views` should be a URL pointing to the schema or contract document for that view. The referenced document format is implementation-defined and may use JSON, YAML, XML, HTML, or another compatible machine-readable format.

Example:

```json
"views": {
  "blueprint_canvas": "https://architect.example.com/schemas/views/blueprint-canvas.json"
}



### `state_mode`

The continuity model supported by the persona.

Allowed values:

* `stateless`: each interaction is independent
* `session`: the persona supports explicit session continuity
* `hybrid`: the persona supports both stateless and session-based interaction

### `capabilities`

A list of tags or identifiers describing the persona’s declared skills or functions.

Example:

```json
"capabilities": [
  "structural_validation",
  "blueprint_parsing",
  "echo_test"
]
```

In Draft 0.1, capability names are free-form strings and are advisory rather than globally standardized.

### `trust_metadata`

A structure containing public verification and trust-related metadata.

Example:

```json
"trust_metadata": {
  "verified_domain": true,
  "public_key": "pub_xyz123456789"
}
```

Current fields include `verified_domain` and `public_key`. Future versions may extend this model to include signatures, attestations, digests, and provenance metadata.

---

## 6. JSON Schema

This schema defines the required and optional fields for a valid Personafile.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Personafile",
  "description": "The standard public descriptor for a digital persona.",
  "type": "object",
  "required": [
    "spec_version",
    "identity",
    "title",
    "endpoints",
    "state_mode"
  ],
  "properties": {
    "spec_version": {
      "type": "string",
      "description": "The Personafile specification version."
    },
    "identity": {
      "type": "string",
      "format": "uri",
      "pattern": "^pxp://",
      "description": "The unique pxp:// address of the persona."
    },
    "title": {
      "type": "string",
      "description": "A human-readable name for the persona."
    },
    "description": {
      "type": "string",
      "description": "A short summary of the persona's purpose."
    },
    "owner": {
      "type": "string",
      "description": "The entity or authority responsible for the persona."
    },
    "version": {
      "type": "string",
      "description": "The version of the persona descriptor or published persona profile."
    },
    "authentication": {
      "type": "array",
      "description": "A list of supported authentication methods.",
      "items": {
        "type": "string"
      }
    },
    "endpoints": {
      "type": "object",
      "description": "The URLs where the persona accepts PXP interactions.",
      "properties": {
        "message": {
          "type": "string",
          "format": "uri",
          "description": "The endpoint for sending standard PXP message envelopes."
        },
        "event": {
          "type": "string",
          "format": "uri",
          "description": "The endpoint for subscribing to or receiving PXP event envelopes."
        }
      },
      "required": ["message"]
    },
    "views": {
      "type": "object",
      "description": "A mapping of shared interaction view names to external schema or contract documents defining the structured state shared between a persona and a client.",
      "additionalProperties": {
        "type": "string",
        "format": "uri",
        "description": "A URL pointing to the schema or contract document for this view. The document format is implementation-defined and may be JSON, YAML, XML, HTML, or another compatible format."
      }
    },
    "state_mode": {
      "type": "string",
      "enum": ["stateless", "session", "hybrid"],
      "description": "The continuity model the persona supports."
    },
    "capabilities": {
      "type": "array",
      "description": "A list of tags or identifiers defining the persona's skills.",
      "items": {
        "type": "string"
      }
    },
    "trust_metadata": {
      "type": "object",
      "description": "Public verification data for the persona.",
      "properties": {
        "verified_domain": {
          "type": "boolean"
        },
        "public_key": {
          "type": "string"
        }
      }
    }
  }
}
```
---

## 7. Example Personafile

This is an example of a valid `personafile.json` document.

```json
{
  "spec_version": "0.1",
  "identity": "pxp://architect.example.com",
  "title": "Reference Builder Persona",
  "description": "A structural validation persona capable of analyzing architectural plans.",
  "owner": "Internet of Personas Project",
  "version": "0.1.0",
  "authentication": ["none", "signature"],
  "endpoints": {
    "message": "https://architect.example.com/api/v1/pxp/message",
    "event": "wss://architect.example.com/api/v1/pxp/events"
  },
  "views": {
  "type": "object",
  "description": "A mapping of shared interaction view names to external schema or contract documents defining the structured state shared between a persona and a client.",
  "additionalProperties": {
    "type": "string",
    "format": "uri",
    "description": "A URL pointing to the schema or contract document for this view. The document format is implementation-defined and may be JSON, YAML, XML, HTML, or another compatible format."
  },
  "state_mode": "session",
  "capabilities": [
    "structural_validation",
    "blueprint_parsing",
    "echo_test"
  ],
  "trust_metadata": {
    "verified_domain": true,
    "public_key": "pub_xyz123456789"
  }
}
```

---

## 8. Validation Rules

A conforming Personafile should follow these rules:

1. The document **must** be valid JSON.
2. The document **must** include all required fields.
3. The `identity` field **must** be a URI using the `pxp://` scheme.
4. The `endpoints.message` field **must** be present.
5. The `state_mode` field **must** be one of:

   * `stateless`
   * `session`
   * `hybrid`
6. If present, `endpoints.event` **should** describe a valid event delivery endpoint.
7. If present, `capabilities` values **may** be implementation-specific.
8. If present, `trust_metadata` values are public declarations and may require external verification.

---

## 9. Notes on Interoperability

Draft 0.1 intentionally keeps the model compact. Some fields remain intentionally simple:

* `capabilities` are not yet bound to a universal vocabulary.
* `authentication` is not yet a formally registered enum set.
* `trust_metadata` is minimal and will likely expand in future drafts.
* `endpoints` currently identify locations, not full transport contracts.

This is a deliberate design choice for early implementation and experimentation.

---

## 10. Future Extensions

Future drafts of the Personafile Specification may define:

* richer endpoint metadata,
* transport declarations,
* formal authentication profiles,
* signature and digest fields,
* registry attestation structures,
* controlled capability vocabularies,
* localization metadata,
* policy and usage constraints,
* persona icon and presentation metadata.

---

## 11. Relationship to IoP and PXP

The Personafile is part of the broader **Internet of Personas (IoP)** initiative.

Within IoP:

* **IoP** is the overall project and specification family.
* **PXP** is the Persona Exchange Protocol for persona interaction.
* **Personafile** is the public descriptor that enables discovery and interaction bootstrapping.

The Personafile is typically the first document a client or registry reads before interacting with a persona.

---

## 12. Conclusion

The Personafile is the foundation of public persona interoperability on the Internet of Personas.

It gives each persona a standard, machine-readable public descriptor that clients can resolve, inspect, and use to begin interaction. By keeping the model simple and web-native, the Personafile establishes a practical base for discovery, trust, and cross-runtime persona communication.

Draft 0.1 is intended as a working foundation for future refinement.

---

## 13. Open Questions

The following topics remain open for future discussion:

* Should `@context` return in a future JSON-LD-compatible version?
* Should `authentication` become a normative registry?
* Should `trust_metadata.public_key` require a specific encoding format?
* Should endpoint declarations include explicit transport and content-type metadata?
* Should capabilities evolve toward standardized namespaces?

These questions are left open intentionally to support iterative development.

---
