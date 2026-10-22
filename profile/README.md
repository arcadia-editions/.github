# Arcadia Editions

Arcadia Editions is a fictional specialty retailer for board game enthusiasts and tabletop collectors. They curate and sell limited edition board games, premium collector sets, and exclusive collaborations with independent game designers and illustrators.

They operate physical stores in Barcelona, Amsterdam, Tokyo, and Austin. Each store is part shop, part community play space. Online they serve a global tribe of collectors who follow drops, join waitlists, and pre-order months in advance.

Scarcity is intentional. Arcadia Editions does not restock. When a limited edition sells out, it is gone.

**Arcadia Editions is not a real company.** It was created as a showcase for the [ZenWave Platform](https://zenwave360.io), a set of tools for designing, generating, and governing event-driven architectures using AsyncAPI, Kafka, and Spring Boot.

The architecture of Arcadia Editions is documented as a series of blog posts at [ivangsa.com](https://ivangsa.com). Technical deep-dives live at [zenwave360.io](https://zenwave360.io).

[![Event Storming: events, commands, policies and hotspots of the Place Order flow](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow/eventstorming-events-commands-policies-hotspots.jpg?raw=true)](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow/eventstorming-events-commands-policies-hotspots.jpg)

---

## What you will find here

This organization contains the full backend architecture of Arcadia Editions, modeled domain-first using the ZenWave Platform toolchain.

- **[arcadia-editions-architecture](https://github.com/arcadia-editions/arcadia-editions-architecture)**: The architecture world model — `zenwave-architecture.yml`, the master manifest indexing every domain, service and contract, plus cross-domain business flows (ZFL), architecture decision records, and the published Maven packages built from those artifacts. Start here.
- **[api-product-workflows](https://github.com/arcadia-editions/api-product-workflows)**: The reusable GitHub Actions workflows every `*-api` repository calls into for CI validation, artifact releases, and Kafka/Schema Registry provisioning via Terraform — generated straight from each repo's AsyncAPI contracts.
  - [Run Kafka provisioning on your laptop](https://github.com/arcadia-editions/api-product-workflows/blob/main/docs/run-kafka-provisioning-locally.md): generate Terraform from the AsyncAPI contracts of `orders-checkout-api`, plan and apply topics, schemas and ACLs on Confluent Cloud from your own machine, then change a contract and see how the plan changes.
- **[api-contract-commons](https://github.com/arcadia-editions/api-contract-commons)**: Master YAML files for reusable OpenAPI and AsyncAPI sections, including environment servers, common headers, security schemes, protocol bindings, and message traits.
- **[arcadia-event-catalog](https://github.com/arcadia-editions/arcadia-event-catalog)**: The generated EventCatalog site, built from the architecture manifest and published at [arcadia-editions.github.io/arcadia-event-catalog](https://arcadia-editions.github.io/arcadia-event-catalog/).
- **`{domain}-{service}-api`**: AsyncAPI and OpenAPI specs with ZDL domain models. The source of truth for every service contract, built and released through `api-product-workflows`. See [The domain](#the-domain) for the full list.
- **`{domain}-{service}-service`**: Spring Boot microservices generated and extended from the API specs. Planned, except for the [Spring Modulith implementation](#reference-implementation-spring-modulith).


---

## The Place Order flow

The flow was discovered through Event Storming. Click an image to see it full size.

| Events, commands, policies and hotspots | Systems |
|---|---|
| [![Event Storming: events, commands, policies and hotspots of the Place Order flow](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow/eventstorming-events-commands-policies-hotspots.jpg?raw=true)](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow/eventstorming-events-commands-policies-hotspots.jpg) | [![Event Storming: systems of the Place Order flow](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow/eventstorming-systems.png?raw=true)](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow/eventstorming-systems.png) |

The architecture repository models cross-domain business flows in ZFL. The most complete one is [`place-order-flow.zfl`](https://github.com/arcadia-editions/arcadia-editions-architecture/blob/main/business-flows/place-order-flow.zfl), the lifecycle of an order during a limited edition drop. It involves five services: **Orders Checkout**, **Catalog Inventory**, **Payments Processing**, **Fulfillment Shipping** and **Notifications Consumer**.

The flow has several alternative paths: stock unavailable, payment declined, payment retries, fulfillment failure, reservation expiry. Every failure path ends in compensation: the payment authorization is voided, the stock goes back to the pool, and the order is cancelled.

The happy path:

- The customer starts the checkout (`StartOrderCheckout`).
- Orders Checkout asks Catalog Inventory to reserve the stock.
- If the reservation succeeds, the order is created in status `CREATED` and the `OrderCreated` domain event is emitted.
- `OrderCreated` starts the payment authorization in Payments Processing, which places a hold on the customer's funds without moving money.
- When the payment is authorized (`PaymentAuthorized`), the order is confirmed, the stock reservation is confirmed, and `OrderConfirmed` is emitted.
- `OrderConfirmed` starts the fulfillment in Fulfillment Shipping.
- When the fulfillment is scheduled (`FulfillmentScheduled`), Payments Processing captures the payment (`PaymentCaptured`). Money moves only at this point.
- Notifications Consumer sends the order confirmation to the customer (`OrderConfirmationSent`).

Stock is reserved before payment because, in a scarcity-driven drop, inventory is the binding constraint: hold the item first, charge second.

---

## The domain

Six bounded contexts, discovered through Event Storming and modeled in ZDL. Each one has its API defined; the microservices are planned.

| Context | Responsibility | API | Service |
|---|---|---|---|
| Customer & Identity | Profiles, loyalty tiers, authentication | — | — |
| Catalog & Inventory | Products and drops | [`catalog-products-api`](https://github.com/arcadia-editions/catalog-products-api) | Planned |
| Catalog & Inventory | Stock, scarcity enforcement | [`catalog-inventory-api`](https://github.com/arcadia-editions/catalog-inventory-api) | Planned |
| Orders | Cart, checkout, order lifecycle | [`orders-checkout-api`](https://github.com/arcadia-editions/orders-checkout-api) | Planned |
| Payments | Authorization, capture, refunds | [`payments-processing-api`](https://github.com/arcadia-editions/payments-processing-api) | Planned |
| Fulfillment | Picking, packing, shipping, delivery | [`fulfillment-shipping-api`](https://github.com/arcadia-editions/fulfillment-shipping-api) | Planned |
| Notifications | Email, SMS, push — reacting to everything else | [`notifications-consumer-api`](https://github.com/arcadia-editions/notifications-consumer-api) | Planned |

The whole domain is already implemented in the [Spring Modulith](#reference-implementation-spring-modulith) services.

## Why this domain

Scarcity plus global demand plus omnichannel operations. Every product drop, every waitlist update, every sold-out notification is an event. The business demands event-driven architecture. That makes Arcadia Editions a realistic and honest showcase for what the ZenWave Platform does.

---

## Reference implementation: Spring Modulith

**[arcadia-editions-modulith-service-java](https://github.com/arcadia-editions/arcadia-editions-modulith-service-java)** and **[arcadia-editions-modulith-service-kotlin](https://github.com/arcadia-editions/arcadia-editions-modulith-service-kotlin)** are the finished implementations of Arcadia Editions. The same Spring Boot modular monolith, in Java and in Kotlin, implements the whole domain as an event-driven architecture, with the same patterns as the Kafka and microservices design: domain events, commands and policies from the Place Order flow, and the contracts defined in [arcadia-editions-modulith-api](https://github.com/arcadia-editions/arcadia-editions-modulith-api) (OpenAPI and AsyncAPI).

Both are generated from the ZDL domain model and the API specs, and completed with the zenwave-sdd agent skills. Start with their READMEs.

---

## Planned: microservices implementation of the same flow

The next step is to implement the Place Order flow as independent Spring Boot microservices, one per `*-api` repository above, communicating through Kafka with the topics, schemas and ACLs provisioned from their AsyncAPI contracts. Same domain model, same events and commands as the modulith.

---

## Guides

Hands-on guides you can run from your own machine against the public repositories.

| Guide | What you get |
|---|---|
| [Run Kafka provisioning on your laptop](https://github.com/arcadia-editions/api-product-workflows/blob/main/docs/run-kafka-provisioning-locally.md) | Generate Terraform from `orders-checkout-api`'s AsyncAPI contracts, plan and apply topics, schemas and ACLs on Confluent Cloud, then change the contract and read the plan. |
| [Spring Modulith service in Java](https://github.com/arcadia-editions/arcadia-editions-modulith-service-java#readme) | A Java and Spring Boot modular monolith generated from a ZDL domain model, with OpenAPI and AsyncAPI code generated at build time, and the remaining behavior completed with the zenwave-sdd agent skills and GitHub Spec Kit. |
| [Spring Modulith service in Kotlin](https://github.com/arcadia-editions/arcadia-editions-modulith-service-kotlin#readme) | The same service in Kotlin and Spring Boot, from the same domain model and APIs. |

---

## The ZenWave Platform

ZenWave is an open-source platform for designing, developing and governing event-driven architectures.

- **ZDL**: domain modeling language for bounded contexts, aggregates, services and domain events
- **ZFL**: flow modeling language for capturing Event Storming findings
- **ZenWave SDK**: CLI toolkit that generates multiple software artifacts from domain models and APIs, including complete Spring Boot applications
- **ZenWave API-Ops**: AsyncAPI to Terraform generator for keeping your APIs and infrastructure in sync
- **EventCatalog generator**: publishes the full event landscape for visibility — see it live at [arcadia-editions.github.io/arcadia-event-catalog](https://arcadia-editions.github.io/arcadia-event-catalog/)

Learn more at [zenwave360.io](https://zenwave360.io) and [github.com/ZenWave360](https://github.com/ZenWave360).

---

*Built by [Iván García Sainz-Aja](https://www.linkedin.com/in/ivangarciasainzaja/) - ZenWave founder, AsyncAPI Ambassador and TSC member.*
