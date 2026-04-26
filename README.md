# Arcadia Editions

Arcadia Editions is a fictional specialty retailer for board game enthusiasts and tabletop collectors. They curate and sell limited edition board games, premium collector sets, and exclusive collaborations with independent game designers and illustrators.

They operate physical stores in Barcelona, Amsterdam, Tokyo, and Austin. Each store is part shop, part community play space. Online they serve a global tribe of collectors who follow drops, join waitlists, and pre-order months in advance.

Scarcity is intentional. Arcadia Editions does not restock. When a limited edition sells out, it is gone.

**Arcadia Editions is not a real company.** It was created as a showcase for the [ZenWave Platform](https://zenwave360.io) — a set of tools for designing, generating, and governing event-driven architectures using AsyncAPI, Kafka, and Spring Boot.

---

## What you will find here

This organization contains the full backend architecture of Arcadia Editions, modeled domain-first using the ZenWave Platform toolchain.

- **[arcadia-editions-docs](https://github.com/arcadia-editions/arcadia-editions-docs)** — Event Storming findings, ZFL flows, architecture diagrams. Start here.
- **`{domain}-{service}-api`** — AsyncAPI and OpenAPI specs with ZDL domain models. The source of truth for every service contract.
- **`{domain}-{service}-service`** — Spring Boot microservices generated and extended from the API specs.

---

## The domain

Six bounded contexts, discovered through Event Storming and modeled in ZDL:

| Context | Responsibility |
|---|---|
| Customer & Identity | Profiles, loyalty tiers, authentication |
| Catalog & Inventory | Products, drops, stock, scarcity enforcement |
| Orders | Cart, checkout, order lifecycle |
| Payments | Authorization, capture, refunds |
| Fulfillment | Picking, packing, shipping, delivery |
| Notifications | Email, SMS, push — reacting to everything else |

---

## Why this domain

Scarcity plus global demand plus omnichannel operations cannot be handled well with synchronous REST APIs. Every product drop, every waitlist update, every sold-out notification is an event. The business demands event-driven architecture. That makes Arcadia Editions a realistic and honest showcase for what the ZenWave Platform does.

---

## The ZenWave Platform

ZenWave is an open-source platform for event-driven architecture governance and code generation.

- **ZDL** — domain modeling language for bounded contexts and aggregates
- **ZFL** — flow modeling language for capturing Event Storming findings
- **ZenWave SDK** — generates AsyncAPI specs, Spring Boot producers and consumers, and more
- **EventCatalog generator** — publishes the full event landscape for visibility

Learn more at [zenwave360.io](https://zenwave360.io) and [github.com/ZenWave360](https://github.com/ZenWave360).

---

## Follow the story

The architecture of Arcadia Editions is documented as a series of blog posts at [ivangsa.com](https://ivangsa.com). Technical deep-dives live at [zenwave360.io](https://zenwave360.io).

*Built by [Iván García Sainz-Aja](https://github.com/ivangsa) — AsyncAPI Ambassador, ZenWave founder.*
