# Software Architecture Patterns

This subdirectory contains my notes and resources of the [Udacity Software Architecture Patterns Course](https://www.udacity.com/course/software-architecture-patterns--cd14601).

I drew the diagrams using [Excalidraw](https://excalidraw.com/).

Table of contents:

- [Software Architecture Patterns](#software-architecture-patterns)
  - [Introduction to Software Architecture](#introduction-to-software-architecture)
    - [What is Software Architecture?](#what-is-software-architecture)
      - [Example: Food delivery service architecture, with increasing detail](#example-food-delivery-service-architecture-with-increasing-detail)
    - [Why Design Our Own Software?](#why-design-our-own-software)
      - [Examples of Software Design](#examples-of-software-design)
    - [Architecture Pattern: Monolith](#architecture-pattern-monolith)
    - [Architecture Pattern: MVC = Model-View-Controller](#architecture-pattern-mvc--model-view-controller)
      - [Django's MVC Implementation](#djangos-mvc-implementation)
    - [Architecture Pattern: Layered](#architecture-pattern-layered)
      - [Extra: Domain-Driven Design (DDD) and Onion Architecture](#extra-domain-driven-design-ddd-and-onion-architecture)
    - [Architecture Pattern: Event-Driven](#architecture-pattern-event-driven)
    - [Exercise: Architecture Archetypes](#exercise-architecture-archetypes)
      - [Stage 1: Launch the MVP](#stage-1-launch-the-mvp)
      - [Stage 2: Evolve for Growth](#stage-2-evolve-for-growth)
    - [Summary: Introduction to Software Architecture](#summary-introduction-to-software-architecture)
  - [Communicating Software Architecture](#communicating-software-architecture)
    - [Communicating Software Architecture: Introduction](#communicating-software-architecture-introduction)
    - [Mermaid](#mermaid)
    - [D2: Declarative Diagramming](#d2-declarative-diagramming)
    - [Extra: Excalidraw](#extra-excalidraw)
    - [Communicating Sequences](#communicating-sequences)
    - [Sequence Diagrams: Exercise](#sequence-diagrams-exercise)
    - [Communicating Events](#communicating-events)
    - [Event Diagrams: Exercise](#event-diagrams-exercise)
    - [System Architecture](#system-architecture)
    - [System Architecture: Exercise](#system-architecture-exercise)
      - [HVAC System Sequence Diagram](#hvac-system-sequence-diagram)
      - [HVAC System Event Flow Diagram](#hvac-system-event-flow-diagram)
      - [HVAC System Architecture Diagram](#hvac-system-architecture-diagram)
  - [Microservices and Modular Design Patterns](#microservices-and-modular-design-patterns)
    - [From Systems to Services](#from-systems-to-services)
    - [Form Systems to Services: Exercise](#form-systems-to-services-exercise)
    - [Microservices: The Good, The Bad, and The Ugly](#microservices-the-good-the-bad-and-the-ugly)
    - [Exercise: Monoliths vs. Microservices](#exercise-monoliths-vs-microservices)
    - [How Do I Make My Service Micro?](#how-do-i-make-my-service-micro)
    - [Communicating Microservices](#communicating-microservices)
    - [Exercise: Moving to Microservices](#exercise-moving-to-microservices)
    - [Microservice Patterns: Resilience](#microservice-patterns-resilience)
    - [Microservice Patterns: Consistency](#microservice-patterns-consistency)
    - [Microservice Patterns: Performance](#microservice-patterns-performance)
    - [Microservice Patterns: Exercise](#microservice-patterns-exercise)
  - [Cloud-Native Architectural Patterns](#cloud-native-architectural-patterns)
  - [Data and AI Architecture Patterns](#data-and-ai-architecture-patterns)
  - [Distributed Architecture Patterns](#distributed-architecture-patterns)
  - [Project: Real-Time Financial Fraud Detection -- Solution Design](#project-real-time-financial-fraud-detection----solution-design)

## Introduction to Software Architecture

### What is Software Architecture?

- Architectural thinking starts by identifying a system's key components and how they interact.
- We refine a high-level model by adding detail progressively:
  - Identify the actors and external systems: a food delivery platform connects users, drivers, restaurants, and a payment processor.
  - Describe how users access the platform, the functionality it provides, and how it persists data.
  - Specify how requests move between components and how security and performance features, such as rate limiting, work.
- We prioritize architectural characteristics according to the system's needs; for instance:
  - **Banking systems:**
    - **Security:** Protect money and sensitive financial information from unauthorized access.
    - **Availability:** Support around-the-clock operation, subject to specific exceptions.
    - **Integrity:** Keep records correct so payments and withdrawals affect accounts by the intended amounts.
  - **Video streaming services:**
    - **Performance:** Deliver large volumes of video to many users globally. Compression can reduce data volume and improve stream quality; Netflix invests in this area.
    - **Scalability:** Accommodate growing demand, including more replicas, connections, and load.
    - **Ease of use:** Make content easy to find and play across televisions, phones, and tablets, including for young users.
  - **Car control systems:**
    - **Safety:** Use fail-safes that prioritize safety over convenience or performance because malfunctions can cause physical harm.
    - **Real-time responsiveness:** Respond promptly to inputs such as brake-pedal presses; delays can endanger drivers and other road users.
    - **Reliability:** Tolerate real-world inputs and conditions while remaining available and predictable.

![Key Characteristics](./assets/key_characteristics.png)

#### Example: Food delivery service architecture, with increasing detail

![Food Delivery Architecture](./assets/food_delivery_architecture_0.png)

![Food Delivery Architecture](./assets/food_delivery_architecture_1.jpeg)

![Food Delivery Architecture](./assets/food_delivery_architecture_2.jpeg)

Some components:

- **API gateway:** An entry point for client requests to the platform's API (Application Programming Interface).
  - Routes requests to the appropriate backend service, such as orders or restaurants; if it only does that, it's the same as the **reverse proxy**.
  - However, usually, an API Gateway does more than just routing: it can centralize authentication, logging, and rate limiting.
  - Example: Route `POST /orders` from the customer app to the order service. The service still checks business rules, such as whether the customer may access a particular order.
- **Load balancer:** Distributes incoming traffic across healthy server or service instances to spread work and maintain availability.
  - Example: Distribute order requests among three instances of the order service and stop sending requests to an instance that fails health checks.
  - A gateway manages API routes and policies; a load balancer selects an instance to handle traffic. One product can perform both roles, and their placement depends on the architecture.
- **Payment gateway:** Securely exchanges payment information and authorization requests between the application and payment-processing systems.
  - Example: During checkout, submit a payment request and receive its status before advancing the order's payment workflow.
  - A payment provider may bundle gateway and processing functions. The platform's payment service handles its own business rules and records the payment outcome.
- **Redis cache:** A fast, in-memory store used here to keep temporary copies of frequently requested data and reduce database reads.
  - Example: Check Redis for a restaurant menu; on a cache miss, read it from the database and cache the result.
  - Use expiration through a TTL (Time To Live) and update or invalidate cached entries when necessary. Cached data can become stale; the durable database remains the source of truth in this design.
- **Rate limiter:** Restricts how many requests a caller can make within a time period to control abuse and protect system capacity.
  - Example: Allow an illustrative limit of 100 restaurant-search requests per user per minute, then reject excess requests, commonly with HTTP (Hypertext Transfer Protocol) status `429 Too Many Requests`.
  - It can run in the API gateway or a service and use shared Redis counters so multiple instances enforce a common limit.
- **Core services:** Implement the platform's main business capabilities and rules.
  - Examples include managing users, restaurant menus, orders, payments, and driver assignments.
  - They coordinate state changes and integrations: an order service can check availability, request payment, and arrange delivery. These capabilities may be modules in one application or separately deployed services.
- **Data layer:** Stores and retrieves application data through databases and the code that accesses them.
  - Example: Persist customers, restaurants, orders, and payment references so records survive application restarts.
  - Its design determines how data is queried, kept consistent, and recovered. Cache use complements durable storage.
- **Maps API:** An external interface for mapping capabilities, such as displaying maps, converting addresses to coordinates, or calculating routes and travel times.
  - Example: Locate a delivery address and estimate a driver's journey from the restaurant to the customer.
  - Live driver tracking also requires the driver's app to send location updates; route calculations alone do not supply those updates.
- **Push notification service:** Sends messages from the backend to users' devices through platform delivery systems, including when the app is not open on screen.
  - Example: Notify the customer that an order was accepted or that the driver is approaching.
  - Providers such as Firebase Cloud Messaging deliver messages to registered app instances. Keep order status in the backend so opening the app retrieves the current state independently of a notification.

Component references:

- [API gateways and load balancing (Apache APISIX)](https://apisix.apache.org/learning-center/api-gateway-vs-load-balancer/)
- [Payment gateways (Stripe)](https://stripe.com/resources/more/payment-gateways-101)
- [Redis caching](https://github.com/redis/docs/blob/main/content/develop/use-cases/cache-aside/go/_index.md)
- [Redis rate limiting](https://github.com/redis/docs/blob/main/content/commands/incr.md)
- [Google Maps documentation](https://developers.google.com/maps/documentation/)
- [Firebase Cloud Messaging architecture](https://firebase.google.com/docs/cloud-messaging/fcm-architecture)

Additional resource: [Encoding & Quality (Netflix Research)](https://research.netflix.com/research-area/video-encoding-and-quality) explains Netflix's work on video encoding and quality.

### Why Design Our Own Software?

- Choose an architecture that fits current requirements and can evolve as needs change.
- **Twitter example:** Traffic during the 2010 World Cup exposed scaling limits that adding servers could no longer solve economically. Twitter redesigned its stack and reorganized its engineering teams.
  - Its original design favored a quick launch; needing a redesign later did not make that initial trade-off wrong.
  - **Conway's law:** Systems tend to reflect team structure; architectural changes can also drive team restructuring.
- Make trade-offs deliberately: prioritize reliability for safety-critical systems, rapid iteration for early products, and compatibility and data integrity for legacy rewrites.

Further reading: [New Tweets per second record, and how!](https://blog.x.com/engineering/en_us/a/2013/new-tweets-per-second-record-and-how).

#### Examples of Software Design

- **Nuclear power plant control system:** Prioritize stability, safety, and timely responses; accept higher operating costs to maintain these characteristics.
- **Growing social media app:** Prioritize rapid development, adaptability, and a responsive interface; some reduction in reliability may be acceptable initially.
- **Government agency's legacy system rewrite:** Prioritize backward compatibility and reliability to preserve internal workflows; accept greater development effort.
- Balance competing characteristics according to business needs. Improving one does not always require sacrificing another, but some compromises are necessary.

### Architecture Pattern: Monolith

- **A monolith packages the application's business capabilities into one deployable unit**, typically maintained in a shared codebase.
  - Components within the same process communicate through function calls, avoiding network latency and serialization overhead between those components.
  - A single database is common, but is not a requirement of the pattern.
- **Internal structure still matters:** Organize code into clear modules rather than allowing dependencies to grow ad hoc.
  - A layered structure separates presentation, business logic, and data access.
  - MVC (Model–View–Controller) separates data and domain behavior, presentation, and request handling.
- **Advantages:**
  - Simple deployment and local development help small teams ship features, gather feedback, and iterate quickly.
  - Debugging and tracing requests are easier when the relevant code runs in one process.
  - Operations within one transactional database can maintain consistency without coordinating distributed transactions.
- **Limitations as the system grows:**
  - A faulty component can affect the whole application, and deploying a change requires deploying the application as a unit.
  - Scaling usually replicates the entire application, even when only one feature needs more capacity--for example, a busy product catalog alongside rarely used administration features.
  - Shared code and release schedules can increase coordination costs and limit team independence.
- **Good fit:** Startups validating an idea, small teams with limited resources, and applications that benefit from straightforward transactions.
- **Reconsider the design** when deployments become bottlenecks, components need different scaling strategies, or teams need independent releases. A well-structured monolith can still support substantial scale; growth alone does not require microservices.

![Monolithic vs Microservices](./assets/monolith_vs_microservices.png)

![Monolithic vs Ad-hoc](./assets/monolith_vs_adhoc.jpeg)

### Architecture Pattern: MVC = Model-View-Controller

- **MVC (Model–View–Controller)** separates responsibilities within an application:
  - **Model:** Represents domain data, behavior, and business rules independently of presentation.
  - **View:** Presents information to the user, such as a page built with HTML (Hypertext Markup Language) and styled with CSS (Cascading Style Sheets).
  - **Controller:** Handles user input, invokes model operations, and selects the view and data to return.
- **Example -- Add to cart:** The controller receives the request, asks the model to update the cart, and supplies the updated data to the view.
- **Benefits:** Clear responsibilities improve testability, support parallel work, and give developers predictable places for code. Web frameworks adapt these ideas with different terminology.
- **Scope:** MVC organizes application code; a monolith describes how the application is packaged and deployed. They can be used together.
- **Communication differs from strict layering:** MVC does not impose an adjacent-layer-only call sequence. Interactions can connect any of its three roles as the MVC variant permits--for example, a controller updates the model, a view reads model state, and model changes notify the view.
  - This flexibility still preserves responsibilities: the model should remain independent of presentation details, and framework conventions may restrict interactions.
  - Strict layered architecture restricts calls to the next lower layer; relaxed layering may allow skipping layers. MVC can itself organize the presentation layer of a larger layered application.
- **Limitations:** Controllers can become bloated and responsibilities can blur. Related presentation patterns include MVP (Model–View–Presenter) and MVVM (Model–View–ViewModel).

![MVC](./assets/mvc.png)

![MVC Evolution](./assets/mvc_evolution.png)

Further reading: [MVC Explained](https://www.tomdalling.com/blog/software-design/model-view-controller-explained/).

#### Django's MVC Implementation

- Django describes its organization as **MTV (Model–Template–View)**. Its names do not map exactly to conventional MVC terminology. [Official explanation](https://docs.djangoproject.com/en/stable/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names).

| Responsibility | Django implementation |
| --- | --- |
| Model: data and domain behavior | Classes in `models.py`; the ORM (Object–Relational Mapper) provides database access. |
| Presentation, usually called the MVC view | Templates, commonly under `templates/`, define how the selected data appears. |
| Select data and construct a response | A function or class-based view in `views.py` handles a request and may render a template. |
| Dispatch input to a handler | The framework uses URL (Uniform Resource Locator) patterns in `urls.py` to select a view. Django identifies this routing machinery as its controller. |

- So in comparison to the MVC:

| Traditional MVC | Django |
|---|---|
| Model | Model |
| View | Template |
| Controller | View + framework routing (`urls.py`) |

- For a practical comparison, a Django view performs much of the orchestration commonly assigned to a web MVC controller, while routing is handled by the framework.
- **Example -- Display a restaurant menu:** In an existing Django project, register an app named `menu` in `INSTALLED_APPS`, retain the default app-template discovery, and create the following files. Models define fields and behavior; migrations create the corresponding database structure. [Model documentation](https://docs.djangoproject.com/en/stable/topics/db/models/).

```python
# menu/models.py
from django.db import models


class Dish(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=8, decimal_places=2)
```

```python
# menu/views.py
from django.shortcuts import render

from .models import Dish


def menu_list(request):
    dishes = Dish.objects.order_by("name")
    return render(request, "menu/list.html", {"dishes": dishes})
```

```python
# menu/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path("", views.menu_list, name="menu-list"),
]
```

```python
# Project URL configuration, e.g. config/urls.py
from django.urls import include, path

urlpatterns = [
    path("menu/", include("menu.urls")),
    # Retain other project routes here.
]
```

```html
<!-- menu/templates/menu/list.html -->
<h1>Menu</h1>
<ul>
  {% for dish in dishes %}
    <li>{{ dish.name }} -- {{ dish.price }}</li>
  {% empty %}
    <li>No dishes available.</li>
  {% endfor %}
</ul>
```

```bash
python manage.py makemigrations menu
python manage.py migrate
python manage.py runserver
```

- Open `http://127.0.0.1:8000/menu/`: URL routing selects `menu_list`, the view retrieves dishes through the model, and `render()` combines the template and context into an HTTP (Hypertext Transfer Protocol) response. The template displays the empty message until dishes are added. [Routing, views, and templates tutorial](https://docs.djangoproject.com/en/stable/intro/tutorial03/).
- Keep presentation in templates and reusable business rules outside request handlers. Models can hold domain behavior; more complex workflows can use separate application services.

### Architecture Pattern: Layered

- **Layered architecture** separates code by responsibility:
  - **Presentation:** Handles user interaction and incoming requests.
  - **Business logic:** Executes business operations and enforces rules, such as pricing and order validation.
  - **Data access:** Reads and writes persistent data.
- **Dependency rules:** In strict layering, a layer uses only the layer immediately below it; relaxed layering permits skipping layers. For example, presentation should not bypass business rules by updating the database directly.
- **Layers are logical boundaries; tiers are deployment boundaries.** All three layers can run in one monolith; separate layers do not require separate servers.
- **Benefits:** Clear boundaries support focused testing, team responsibilities, and reuse of business rules across web and mobile interfaces.
  - Hiding database details behind stable interfaces reduces the impact of switching databases, although differences in queries, transactions, and schemas can still require changes.
- **Costs:** Unnecessary pass-through layers add complexity and call or mapping overhead. Network latency arises when calls cross deployment boundaries, not simply because code has layers.
- **Relationship to MVC (Model–View–Controller):** MVC separates interaction and presentation responsibilities; layering defines broader application boundaries. MVC can organize the presentation layer of a layered application.

![Layered Architecture](./assets/layered_architecture.png)

#### Extra: Domain-Driven Design (DDD) and Onion Architecture

- **DDD (Domain-Driven Design):** Model business concepts and rules with domain experts.
  - **Ubiquitous language:** Shared terminology in discussions and code.
  - **Bounded context:** A boundary within which a model applies, such as ordering or delivery.
  - **Domain objects:** Entities have identity; value objects are defined by values; aggregates enforce consistency through a root entity; domain services hold rules that do not belong to one object.
- **Fit with layering:** Each context can separate these responsibilities:

| Layer | Responsibility | Example |
| --- | --- | --- |
| Presentation | Handle input and responses. | Receive an order request. |
| Application | Coordinate the use case. | Load an order, call `accept()`, save it. |
| Domain | Enforce business rules. | Reject acceptance of a cancelled order. |
| Infrastructure | Implement storage and integrations. | Save orders; send notifications. |

- **Onion architecture:** Source-code dependencies point inward toward the domain. Inner layers define interfaces; infrastructure implements them, keeping business rules independent of databases and frameworks. Runtime calls can still reach outward through those interfaces.
- **Use when business rules are complex:** DDD shapes the model; Onion protects it from technical details. Both work within a monolith and do not require microservices.

![Onion Architecture](./assets/onion_architecture.png)

References:

- [Bounded contexts](https://martinfowler.com/bliki/BoundedContext.html)
- [DDD layers](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)
- [Domain model patterns](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/microservice-domain-model)
- [Onion/Clean architecture](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures).

### Architecture Pattern: Event-Driven

- **Event-driven architecture:** Components publish events describing what happened, such as `OrderReceived` or `PaymentFailed`; interested consumers react to them.
  - In **publish–subscribe**, publishers send events through an event channel or broker without calling each subscriber directly.
  - Consumers typically process events asynchronously; the publisher does not wait for their business operations to finish.
- **Example -- Checkout:**
  - With synchronous request–response, checkout waits for payment, inventory, and shipping calls; a slow or unavailable dependency can delay the response.
  - With events, checkout records the order, publishes `OrderReceived`, and acknowledges that processing has started. Payment, inventory, notifications, and analytics react as the workflow progresses.
  - Acknowledging receipt does not mean payment succeeded or delivery is confirmed; dependent steps still need coordination.
- **Benefits:**
  - Loose coupling lets consumers evolve and scale independently.
  - New subscribers, such as fraud detection or marketing services, can react to existing events without changing publisher logic, provided those events contain the information they need.
  - Fits workflows spanning multiple systems or teams.
- **Trade-offs:**
  - **Eventual consistency:** Updates happen at different times; an accepted order may appear before inventory is updated.
  - **Harder tracing:** A workflow crosses handlers and services, so logs must be correlated rather than followed through one call stack.
  - **Failure propagation:** Invalid events or faulty consumers can disrupt downstream processing; asynchronous communication still requires error handling.
- **Fit with other patterns:** A monolith or layered application can also use events. Deployment structure, code organization, and communication style are separate design choices.
- **Use when** asynchronous workflows and loose coupling justify the additional coordination and operational complexity.

![Request-Response](./assets/request_response.png)

![Event-Driven Architecture](./assets/event_driven_architecture.png)

![Comparing Request and Event](./assets/comparing_request_and_event.png)

![Publisher-Subscriber](./assets/publisher_subscriber.png)

![Synchronous vs Asynchronous](./assets/synch_asynch.png)

Further reading: [Event-Driven Architecture -- AWS](https://aws.amazon.com/event-driven-architecture/).

### Exercise: Architecture Archetypes

- **Scenario:** EcoDweller Solutions is building a SHEMS (Smart Home Energy Management System) to help homeowners monitor and optimize energy use. Architecture should evolve with its business needs.

#### Stage 1: Launch the MVP

- **Goal:** A small team with limited funding needs to launch an MVP (Minimum Viable Product) quickly and validate it with early adopters.
- **Requirements:** Ingest real-time readings from one smart-meter model, display historical consumption in a basic web dashboard, and manually switch a smart plug on or off.
- **Proposed architecture:** A monolith handles web requests, authentication, ingestion, dashboard rendering, and device communication; an external database persists the data.
  - **Advantage:** One application simplifies development and deployment, helping the team launch and iterate quickly.
  - **Future risk:** Coupled releases, shared failures, and scaling the whole application can become constraints as usage and features grow.

![EcoTech: Product Goals](./assets/ecotech_product_goals.png)

![Example Monolith Structure with Everything in One Place](./assets/ecotech_monolith.jpeg)

#### Stage 2: Evolve for Growth

- **Context:** Funding, users, and team size have increased; deployments are slowing and bugs can affect the whole application.
- **New requirements:** Consumption-threshold alerts, scheduled device actions, support for two additional device types such as thermostats and solar inverters, and greater resilience to component failures.
- **Proposed evolution:** Extract ingestion and alerting into separate services, connected through a message queue for asynchronous event handling.
  - Extend ingestion for the new device types; retain and extend scheduling and device control in the core application.
  - Keep authentication and other capabilities in the monolith rather than splitting every feature into a service.
  - **Advantage:** Ingestion and alerting can be maintained and deployed independently. Asynchronous processing reduces direct availability dependencies, provided failures and queued work are handled correctly.
  - **Trade-off:** More services and a queue introduce operational overhead, harder tracing, and a need to manage retries, duplicate messages, and delayed processing.
- **Decision principle:** Make incremental changes that address current requirements. Other designs are valid if their benefits and trade-offs can be justified; a complete microservices or event-driven migration is not required.

![EcoTech: New Features](./assets/ecotech_new_features.png)

![Monolith Structure with Peeled Out Microservices](./assets/ecotech_microservices.jpeg)

### Summary: Introduction to Software Architecture

- **Monolith:** A single deployable application keeps development and operations simple, making it a useful starting point for small teams validating an idea.
- **MVC (Model–View–Controller):** Separates domain data and behavior, presentation, and input handling within an application.
- **Layered architecture:** Establishes boundaries between presentation, business logic, and data access to improve maintainability as complexity grows.
- **Event-driven architecture:** Decouples components through events and asynchronous processing, supporting integrations and independent scaling while complicating debugging and consistency.
- **Choose and combine patterns according to current needs:** They address different concerns and can coexist. Added architectural complexity should solve a concrete problem.
- **Document trade-offs and revisit decisions:** A suitable design today may need to evolve as requirements, team size, and operational demands change.

[Video: Summary of Introduction to Software Architecture](https://www.youtube.com/watch?v=04rEKcoSH3I)


## Communicating Software Architecture

### Communicating Software Architecture: Introduction

- **Effective communication makes architectural decisions usable.** Teams must understand decisions to implement and maintain the system correctly.
  - Explain the purpose of each component and the problem it solves.
  - Be clear about complexity and trade-offs.
- **Adapt the explanation to the audience.** The same system needs different levels of detail and focus.
  - **Technical teams:** Implementation details, authentication flows, error handling, database schemas, and dependencies.
    - **Online store example:** An API (Application Programming Interface) gateway leads to REST (Representational State Transfer) endpoints using Spring MVC (Model-View-Controller) and a GraphQL interface.
      - An order facade orchestrates validation, pricing, inventory, and payments.
      - Data access uses REST or gRPC (gRPC Remote Procedure Call) clients, a JPA (Jakarta Persistence API)/Hibernate repository, and an event publisher.
      - Dependencies include user, product inventory, and payment microservices, plus external payment, tax, and shipping services.
      - PostgreSQL stores data, Redis provides caching, and Kafka carries events.
  - **Business stakeholders:** Business value, user journeys, and system capabilities.
  - **Operations teams:** Operational impact, failure modes, and scaling characteristics.
    - **Online store example:** Web, mobile, administrator, and vendor interfaces connect through an API gateway.
      - Core services handle users, products, orders, inventory, carts, and search.
      - Infrastructure includes a message queue, analytics pipeline, monitoring, logging, and a CDN (Content Delivery Network).
      - Data stores include user, order, and product databases, a session cache, a search index, and image storage.
- **Choose diagrams according to the question being answered.**
  - **Sequence diagrams:** Show synchronous interactions step by step, such as a user action triggering connection checks, asset loading, and dependency calls.
    - Event-driven flows with multiple consumers can become difficult to follow in this format.
  - **Data flow or event flow diagrams:** Show how information moves asynchronously between decoupled components.
    - Example: An occupancy sensor publishes data through an event bus to multiple consumers.
  - **Solution architecture diagrams:** Show the overall system, major components, data sources and destinations, and external dependencies.
    - Useful for executive briefings and onboarding new team members.
- **Choose diagramming tools that fit the workflow.**
  - **Visual tools:** Visio suits Microsoft environments, Miro supports collaborative whiteboarding, and draw.io provides a free browser-based option.
    - Useful for quick sketches, real-time collaboration, and presentation-ready diagrams.
  - **Diagram as code:** Mermaid and D2 express diagrams as plain text and generate consistent visuals.
    - Update diagrams by editing text stored alongside code; Git history tracks architectural changes.
    - Generate documentation diagrams through CI/CD (Continuous Integration/Continuous Delivery) pipelines to keep them aligned with the system.

![Sequence Diagrams Example](./assets/sequence_diagrams.png)

![Event Flow Diagrams Example](./assets/event_flow_diagrams.png)

![System Architecture Diagrams Example](./assets/system_architecture_diagrams.png)

![Diagram for Technical Team Members](./assets/diagram_technical.jpeg)

![Diagram for Operations Teams](./assets/diagram_operations.jpeg)

### Mermaid

- **Mermaid creates diagrams from human-readable plain text.** Its syntax remains understandable without rendering.
  - Keep diagram source in Markdown documentation alongside application code.
  - Track changes in version control and review them in pull requests.
  - Embedded source avoids separate binary diagram files, missing originals, and broken links to exported diagrams.
- **Each diagram type has its own syntax with consistent patterns.**
  - **Flowcharts:** `A --> B` connects two nodes.
  - **Sequence diagrams:** `Alice->>Bob: Hello` sends a labeled message.
  - **Class diagrams:** `Vehicle <|-- Car` indicates that `Car` inherits from `Vehicle`.
- **Use a text editor or IDE (Integrated Development Environment) to write diagrams.** A browser or editor plugin is optional for previewing them.
  - The [Mermaid Live Editor](https://mermaid.live/) provides live previews while editing.
  - Embed diagrams in fenced `mermaid` code blocks for supported Markdown renderers, including GitHub, GitLab, Notion, and documentation sites.
  - IDEs such as Visual Studio Code may need an extension; Markdown support alone does not guarantee Mermaid rendering.
  - Rendered diagrams can use SVG (Scalable Vector Graphics).
- **Mermaid prioritizes simplicity and Markdown integration over visual polish.** Try it alongside the next diagramming tool to compare their fit for your workflow.
- **Example: Remotely change an HVAC (Heating, Ventilation, and Air Conditioning) system's temperature.**
  - `flowchart LR` sets a left-to-right layout; square brackets define actions, braces define decisions, and labeled arrows distinguish outcomes.
  - The diagram below shows validation, authorization, availability checks, command execution, and the final user message.

![Mermaid Basics](./assets/mermaid_basics.png)

```mermaid
flowchart LR
    Request[User requests a temperature change] --> Range{Temperature in valid range?}
    Range -->|Yes| Auth{User authorized?}
    Range -->|No| Invalid[Invalid temperature error]
    Auth -->|Yes| Online{Device online?}
    Auth -->|No| Denied[Authorization error]
    Online -->|Yes| Command[Issue temperature command]
    Online -->|No| Offline[Device offline error]
    Command --> Success[Confirmation message]
    Invalid --> Display[Display result to user]
    Denied --> Display
    Offline --> Display
    Success --> Display
```

### D2: Declarative Diagramming

- **D2 (Declarative Diagramming) creates diagrams from text, emphasizing expressiveness and visual polish.**
  - Suitable for publication-quality architecture documentation and executive presentations.
  - Source files remain version controlled alongside code.
  - Offers more styling control than simpler tools, with a steeper learning curve; start with boxes and arrows and refine them incrementally.
- **Use a text editor and the standalone D2 command-line tool.** See the [installation guide](https://d2lang.com/tour/install/) for platform-specific instructions.
  - Installation options include Homebrew on macOS, the shell installer on Linux, Scoop on Windows, and building from source with Go.
  - Verify installation with `d2 --version`.
  - Save source as `diagram.d2` and render it to SVG (Scalable Vector Graphics), PNG (Portable Network Graphics), or PDF (Portable Document Format).
  - Use `--watch` to regenerate diagrams as source changes.
  - Integrate rendering into CI/CD (Continuous Integration/Continuous Delivery) pipelines and static documentation sites.
- **Describe structure and appearance declaratively.**
  - Connections use `frontend -> backend`; add text after a colon to label components or interactions.
  - Curly braces group related components into nested containers.
  - Properties control shapes, fill and stroke colors, fonts, icons, and themes.
  - Images can come from remote HTTP (Hypertext Transfer Protocol) URLs or local files, including SVG assets mixed with built-in shapes.
  - Grids arrange components into rows or columns, with placement inferred from the declared structure and constraints.
- **Select a layout engine with `--layout`.** Available features depend on the engine.
  - **Dagre:** Free, default engine for directed, hierarchical layouts.
  - **ELK:** Free alternative for directed graph layouts.
  - **TALA:** Proprietary Terrastruct engine installed separately, designed for software architecture diagrams.
    - Supports explicit `top` and `left` coordinates to fix object positions while arranging other elements around them.
- **Example: An API (Application Programming Interface) gateway routes requests to backend services.** The examples below show labeled connections and component styling in D2 and Mermaid.

![D2 Syntax Basics](./assets/d2_syntax_basics.png)

![D2 Example](./assets/d2_example.png)

![D2 Rendering](./assets/d2_rendering.png)

![D2 Example 2](./assets/d2_example_2.png)

Installation alternatives; choose the command for your environment:

```sh
# macOS with Homebrew
brew install d2
# Linux with the shell installer
curl -fsSL https://d2lang.com/install.sh | sh -s --
# Windows with Scoop
scoop install d2
# From source (requires Go)
go install github.com/d2lang/d2@latest
```

Save this D2 example as `diagram.d2`:

```d2
direction: right
client: Client
gateway: API Gateway {
  shape: hexagon
  style.fill: "#dbeafe"
  style.stroke: "#2563eb"
}
services: Backend services {
  users: User Service
  orders: Order Service
}
client -> gateway: Request
gateway -> services.users: User requests
gateway -> services.orders: Order requests
```

```sh
d2 --version
d2 diagram.d2 diagram.svg
d2 diagram.d2 diagram.png
d2 diagram.d2 diagram.pdf
d2 --layout elk diagram.d2 diagram.svg
d2 --watch diagram.d2 diagram.svg
```

Equivalent Mermaid example for Markdown rendering:

```mermaid
flowchart LR
    Client[Client] -->|Request| Gateway{{API Gateway}}
    subgraph Services[Backend services]
        Users[User Service]
        Orders[Order Service]
    end
    Gateway -->|User requests| Users
    Gateway -->|Order requests| Orders
    style Gateway fill:#dbeafe,stroke:#2563eb
```

### Extra: Excalidraw

- **Excalidraw is a visual whiteboard for sketching diagrams with a hand-drawn appearance.** Use it for architecture discussions, component overviews, and early design ideas.
- **Install the VS Code (Visual Studio Code) extension.**
  - Open Extensions with `Ctrl+Shift+X`, search for **Excalidraw**, and install the extension with identifier `pomdtr.excalidraw-editor` ([Marketplace listing](https://marketplace.visualstudio.com/items?itemName=pomdtr.excalidraw-editor)).
- **Create a drawing inside the repository.**
  - In Explorer, create an empty file such as `assets/order-flow.excalidraw` and open it.
  - The extension opens an interactive canvas; if another editor opens, right-click the file and use **Open With...** to select Excalidraw.
  - Use rectangles for services, diamonds for decisions, arrows for interactions, and text for labels.
  - Example: Draw `Client --> API Gateway --> Order Service`, then connect the order service to a database.
  - Select elements to adjust colors, borders, and text; group related components to keep the diagram readable.
  - Save with `Ctrl+S` and commit the drawing with the documentation it explains.
- **Choose a format according to how the drawing will be used.**
  - `.excalidraw` is the native file format and stores editable scene data as JSON (JavaScript Object Notation).
  - For a straightforward workflow, keep this source file and use the canvas export menu to export a separate `.svg` or `.png` image for documentation.
  - After changing the source drawing, export the image again and commit both files.
  - Alternatively, the VS Code extension supports `.excalidraw.svg` and `.excalidraw.png`: images with embedded scene data that remain editable, combining source and documentation image in one file.
  - Prefer SVG (Scalable Vector Graphics) for sharp diagrams at different sizes; use PNG (Portable Network Graphics) when a raster image is needed.
  - The editor toolbar lets you switch between the canvas and the underlying text or image.
  - Arbitrary SVG or PNG images do not become editable Excalidraw scenes simply by renaming them; they need embedded scene data.
- **Embed the exported image in Markdown using a relative path.** The example below assumes the drawing is in `assets` beside the README.

```markdown
![Order request flow](./assets/order-flow.svg)
```

Extension reference: [Excalidraw for VS Code](https://github.com/excalidraw/excalidraw-vscode).

### Communicating Sequences

- **Sequence diagrams show behavior unfolding over time:** who interacts, in what order, and with which messages.
  - Vertical lifelines represent participants; horizontal arrows represent interactions.
  - Time progresses from top to bottom, revealing the order of operations.
- **Use them to explain request-response behavior, especially when components wait for replies before continuing.**
  - **API (Application Programming Interface) flows:** Trace authentication, validation, business logic, and data persistence.
  - **User journeys:** Follow a user action across components, such as a button click loading a video from the browser through backend systems to a device and back.
  - **Error handling:** Document authentication failures, invalid inputs, and timeouts alongside the successful path.
  - For asynchronous, event-driven architectures, data flow or event flow diagrams often communicate the overall behavior more clearly.
- **Keep diagrams focused and readable.**
  - Show architecturally significant interactions across component boundaries rather than every method call.
  - Use clear, consistent participant names, such as `User Service` instead of `US` or `Service 1`.
  - Annotate timing constraints, including timeouts, to explain behavior under load.
  - Group related steps with frames or notes; Mermaid's `alt` / `else` blocks express alternative outcomes.
- **Mermaid and D2 both support sequence diagrams.**
  - **Mermaid example below:** A user controls an HVAC (Heating, Ventilation, and Air Conditioning) device through a gateway and central server.
    - JWT means JSON Web Token; HTTP means Hypertext Transfer Protocol.
    - The success response confirms that the temperature change was initiated, rather than that the room reached the requested temperature.
    - `participant ... as ...` defines readable labels; `->>` shows calls, `-->>` shows replies, and `CS->>CS` represents internal validation.
  - **D2 example image:** A mobile app requests a profile through a gateway; an authentication service validates the token against user data and permissions before profile retrieval through a profile service and database. Responses return toward the app.

![When to Use Sequence Diagrams](./assets/sequence_diagrams_when.png)

![Sequence Diagram in Mermaid 1](./assets/sequence_mermaird_1.png)

![Sequence Diagram in Mermaid 2](./assets/sequence_mermeid_2.png)

![D2 Sequence Diagram](./assets/d2_sequence_diagram.jpeg)

Mermaid example transcribed from the two syntax images:

```mermaid
sequenceDiagram
    participant U as User Browser
    participant AG as API Gateway
    participant CS as Central Server
    participant DeviceDB as Device Database
    participant HVAC as HVAC Controller
    U->>AG: POST /api/hvac/temperature {temp: 72, room: "Conference A"}
    AG->>CS: Authenticate request with JWT token
    CS-->>AG: Authentication successful
    AG->>CS: Forward temperature change request
    CS->>DeviceDB: Query device capabilities for room
    DeviceDB-->>CS: Return device ID and temp range
    CS->>CS: Validate temperature within range
    CS->>HVAC: Send temperature command {deviceId: "hvac-001", temp: 72}
    HVAC-->>CS: Acknowledge command received
    CS-->>AG: Success response with command ID
    AG-->>U: HTTP 200: Temperature change initiated
    Note over U,HVAC: Full synchronous flow from user action to device command
```

### Sequence Diagrams: Exercise

- **Task:** Model a registered, logged-in user turning on an exterior door's magnetic lock through a web dashboard in a centralized security and HVAC (Heating, Ventilation, and Air Conditioning) system.
  - Show the synchronous request-response sequence from the user's click to the final response.
  - Prioritize clarity and accuracy; use Mermaid, D2, draw.io, or another graphical editor.
  - Deliver an image, such as PNG (Portable Network Graphics) or SVG (Scalable Vector Graphics), or a text file containing the diagram definition.
- **Required participants and responsibilities:**
  - **User's Browser:** Starts the interaction.
  - **API (Application Programming Interface) Gateway:** Provides the single entry point for client requests and forwards authentication information in a header.
  - **Central Server:** Verifies the user's session and permission to control the door before issuing a device command.
  - **Maglock Device:** Executes the command and reports its status.
- **Solution:** The diagram below shows the successful path, with session and authorization checks preceding the lock command and the resulting status returning through the server and gateway to the browser.
  - HTTP means Hypertext Transfer Protocol; `200` indicates a successful response.

![Sequence Diagram Exercise](./assets/sequence_diagram_exercise.jpeg)

Mermaid equivalent of the solution image:

```mermaid
sequenceDiagram
    participant U as User's Browser
    participant AG as API Gateway
    participant CS as Central Server
    participant M as Maglock Device
    U->>AG: POST /api/doors/exterior/lock
    AG->>CS: Forward request with auth token
    CS->>CS: Verify user session
    CS->>CS: Check authorization for door control
    CS->>M: Send LOCK command
    M->>M: Engage magnetic lock
    M-->>CS: Return status: LOCKED
    CS-->>AG: Success response with status
    AG-->>U: HTTP 200
```

### Communicating Events

- **Event flow diagrams map asynchronous data journeys:** producers publish events, and consumers process them independently, potentially at different times and speeds.
  - Emphasize loose coupling and parallel processing rather than a synchronous conversation in which each component waits for a reply.
  - Independent consumers can isolate failures so one consumer's failure need not stop the others.
- **Use event flow diagrams for:**
  - Publish-subscribe microservices, stream processing, and reactive architectures.
  - Background processing: A file upload triggers virus scanning, thumbnail generation, metadata extraction, and storage concurrently.
  - Messaging infrastructure: Show producers, brokers, topics or queues, and consumers; examples include RabbitMQ, Apache Kafka, AWS (Amazon Web Services) SNS (Simple Notification Service), and SQS (Simple Queue Service).
  - IoT (Internet of Things) networks: Trace sensor readings through ingestion, analytics, and business rules to resulting actions.
    - Example: A temperature event independently triggers HVAC (Heating, Ventilation, and Air Conditioning) adjustments, logging, alerting, and analytics.
- **Make the event relationships explicit.**
  - Label event types meaningfully and show their sources and flow direction; event direction can differ from the order of business-process steps.
  - Group related consumers and show feedback loops where a consumer publishes a new event.
  - Show communication through the event bus without implying direct calls or synchronous waits between services.
- **Common patterns:**
  - **Fan-out:** One event reaches multiple consumers, such as notification, logging, and analytics services.
    - The order example distributes events to inventory, payment, shipping, email, fulfillment, analytics, and audit services.
    - Inventory, payment, and shipping also publish events back to the stream; the diagram connects fulfillment to a customer and analytics to a warehouse system.
  - **Event sourcing:** Persist the history of state-changing events as the source of truth and derive current state by replaying them.
    - Useful for audit trails and complex business processes; an event stream alone does not establish event sourcing without an authoritative event store ([reference](https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)).
  - **Saga:** Coordinate a long-running business process across services through local steps and subsequent events or commands.
    - Compensating actions address completed steps when the workflow fails, helping restore business consistency ([reference](https://learn.microsoft.com/en-us/azure/architecture/patterns/saga)).
  - **CQRS (Command Query Responsibility Segregation):** Separate responsibilities for state-changing commands and data-reading queries.
- **Distinguish workflow coordination from message transport.**
  - **Orchestration:** A central controller directs workflow steps and handles outcomes.
  - **Choreography:** Services react to relevant events and publish their own outcomes without a central workflow controller.
    - Useful across bounded contexts, where each domain boundary should require minimal knowledge of other services.
    - Example: The order service publishes `OrderPlaced`; inventory reacts and publishes `InventoryReserved`; shipping consumes the relevant order and inventory events.
  - A central broker can support either approach; its presence alone does not imply orchestration, despite the labels in the example images ([reference](https://learn.microsoft.com/en-us/azure/architecture/patterns/choreography)).

![Event Flow Diagram](./assets/event_flow_diagrams_2.png)

![Fan-Out Pattern](./assets/fan_out_pattern.png)

![Event Sourcing Pattern](./assets/event_sourcing_pattern.png)

![Saga Pattern](./assets/saga_pattern.png)

![Event Choreography Pattern](./assets/event_choreography.png)

### Event Diagrams: Exercise

- **Scenario:** An occupancy sensor estimates the number of people in a room. The HVAC (Heating, Ventilation, and Air Conditioning) system uses this reading to adjust lighting and air-conditioning targets for staff comfort and records occupancy history to assess room usage.
- **Task:** Create a flowchart or data flow diagram showing the data packet's asynchronous journey from its source to storage.
  - Highlight handoffs between services; physical devices can take time to act, so avoid implying synchronous request-response calls.
  - Use any suitable tool, including draw.io, Mermaid, or D2.
- **Required components and responsibilities:**
  - **Occupancy Sensor:** Produces the original reading.
  - **Device Ingestion Service:** Receives raw device data.
  - **Event Bus:** Brokers messages and decouples services.
  - **Central Controller:** Consumes, validates, and transforms readings; applies preconfigured lighting and air-conditioning targets for the occupancy count; writes historical records.
  - **Time-Series Database:** Stores the processed data over time.
- **Solution approaches:** Choose the view that clearly communicates workflow ownership and event movement to the intended audience.
  - **Orchestration:** A central workflow controller receives inputs, coordinates processing, and uses service outputs to trigger subsequent steps.
    - It may contain business logic; coordination still happens through asynchronous events.
    - The simplified image uses temperature and humidity sensors: the HVAC Workflow Orchestrator sends data to an ML (Machine Learning) Analytics Service, receives results, controls the HVAC Controller, and receives device status.
  - **Choreography:** Workflow logic is distributed among services that react to events and publish further events, including fan-out and feedback loops.
    - The simplified image omits some sensor types to make the flow easier to trace.
    - Temperature and occupancy events feed a Climate Analyzer, which signals cooling requirements to the HVAC Controller, demand spikes to the Energy Monitor, and critical temperatures to the Alert Manager.
    - The HVAC Controller publishes status to the Building Dashboard and activation events to the Energy Monitor.
    - Weather forecasts and occupancy priority updates also feed energy monitoring; usage patterns feed a Predictive Engine, which recommends pre-cooling to the HVAC Controller.
    - Metrics update the dashboard, while alerts reach a Notification Service.
  - A broker can carry events in either style; the distinction is where workflow decisions reside, rather than whether messages pass through centralized infrastructure.
- **Check the exercise requirements against the examples.** The images illustrate broader climate-control workflows; your occupancy diagram must still show ingestion, the event bus, lighting adjustments, and historical time-series storage.

![Simplified Orchestrator Event Flow Diagram](./assets/event_diagram_exercise.jpeg)

![Choreograph-Style Event Flow Diagram](./assets/event_diagram_choreography_exercise.jpeg)

### System Architecture

- **System architecture diagrams map the technology landscape:** the major components, their responsibilities, and their relationships.
  - Provide an overview of what exists and how it fits together; sequence and event flow diagrams describe interactions and event movement in greater detail.
- **Include the architectural information relevant to the audience.**
  - **Components and services:** The system's major building blocks.
  - **Data flow and storage:** Where information lives and how it moves between systems, especially along critical business paths.
  - **External dependencies:** Third-party services, APIs (Application Programming Interfaces), and integrations.
  - **System boundaries:** Ownership and the distinction between components under your control and external systems.
  - **Technology choices:** Databases, messaging systems, and cloud services.
- **Choose the level of detail for the purpose.**
  - **Executive briefings:** Explain capabilities, business objectives, technology strategy, and investment areas.
  - **Onboarding engineers:** Provide a map of the overall landscape and where functions reside before examining individual components.
  - **Integration planning:** Show partners and internal teams available interfaces, dependencies, and integration points.
  - **Roadmaps:** Compare the current architecture with future states for migrations, evolution, and new capabilities.
  - **Compliance and security reviews:** Help auditors and security teams assess the boundaries and data dependencies described above.
- **Use visual structure to make the diagram readable.**
  - Group related components or architectural layers with boxes, colors, or swim lanes.
  - Emphasize critical user journeys and data paths to reveal important dependencies and potential bottlenecks.
  - Keep the view streamlined: include what viewers need for shared understanding without overwhelming them with implementation details.
  - Update diagrams as part of the documentation process and keep them in source control so they evolve with the system.
- **Example: The same system at two levels of detail.**
  - **Basic view:** Backend, load balancer, API, database, and storage.
  - **Advanced view:** An API gateway connects frontends and external integrations to core services, with infrastructure and data stores shown separately.
    - **Frontends:** Web store, mobile app, administrator portal, and vendor portal.
    - **External integrations:** Shipping APIs, tax service, email provider, and fraud detection.
    - **Core services:** Users, products, orders, inventory, carts, and search.
    - **Infrastructure:** Message queue, analytics pipeline, monitoring, logging, and CDN (Content Delivery Network).
    - **Data stores:** User, order, and product databases, a session cache, a search index, and image storage.

![Key Characteristics of System Architecture Diagrams](./assets/system_architecture_diagrams_key_characteristics.jpeg)

![Basic Versus Advanced Architecture Diagrams](./assets/system_architecture_basic_vs_advanced.jpeg)

### System Architecture: Exercise

- **Scenario:** External auditors need one high-level overview of the deployed security and HVAC (Heating, Ventilation, and Air Conditioning) system and its major relationships.
- **Task:** Create a solution architecture diagram, using a component view or a C4-style container view, that distinguishes client requests from asynchronous messaging.
  - Use draw.io, Mermaid, D2, or another suitable tool; keep the overview understandable without excessive technical detail.
  - **Web Dashboard:** Provides user interaction.
  - **API (Application Programming Interface) Gateway:** Supplies the single entry point for client applications.
  - **Device Ingestion Service:** Receives sensor reports.
  - **Event Bus:** Carries asynchronous messages between services.
  - **Central Control Service:** Consumes, validates, and transforms readings; adjusts lighting and air-conditioning targets using preconfigured occupancy rules; writes historical records.
  - **Data stores:** Include at least two database types, such as relational storage for user accounts and time-series storage for sensor data.
- **Suggested architecture:** Devices stream data through a site-local gateway, which aggregates readings and forwards them to ingestion for possible preprocessing and storage.
  - Time-series or analytical storage, such as Prometheus or ClickHouse in the lesson's examples, makes readings available to query services, analytics, and ML (Machine Learning) services.
  - Distinct user personas can have separate interfaces behind the API gateway.
  - External services and data sources enrich collected data and provide integration points.
- **Provided solution samples cover three complementary views.** Alternative designs are valid when they communicate the required behavior and structure clearly.
  - **Sequence:** A facilities manager changes three conference-room targets to 70°F; zone permissions and device mappings are checked, temperatures are validated against 65–75°F, commands run in parallel, and an audit record is created.
    - A separate restricted-zone attempt returns HTTP (Hypertext Transfer Protocol) `403` before device changes.
    - Controller acceptance confirms receipt of the command; it does not establish that a room has reached its target temperature.
  - **Event flow:** Sensor and weather inputs feed an event platform; real-time analytics, historical processing, and prediction operate independently.
    - Climate control, ventilation, alerts, and energy optimization publish outcomes, while dashboards receive live metrics and status updates.
  - **Architecture:** Devices, edge translation and buffering, cloud services, storage, user interfaces, and external integrations form the overall deployed landscape.
    - The sample below adds the required event bus and makes user-account access and historical storage explicit.
- **Image context:** The attached solution image depicts an online store, so use it as a grouping and layout reference rather than an HVAC-specific solution.

![System Architecture Solution](./assets/system_architecture_exercise.jpeg)

#### HVAC System Sequence Diagram

```mermaid
sequenceDiagram
    participant FM as Facilities Manager
    participant WD as Web Dashboard
    participant ZA as Zone API
    participant AS as Authorization Service
    participant ZDB as Zone Database
    participant HC as HVAC Controller
    participant AUD as Audit Service

    FM->>WD: Select zones [Conf-A, Conf-B, Conf-C] set temp to 70°F
    WD->>ZA: POST /api/zones/bulk-temperature {zones: [...], temp: 70}
    ZA->>AS: Verify user permissions for zones
    AS->>AS: Check role and zone access rights
    AS-->>ZA: Authorized for all zones

    ZA->>ZDB: Get zone configurations for [Conf-A, Conf-B, Conf-C]
    ZDB-->>ZA: Zone configs with device mappings
    ZA->>ZA: Validate temperature within range (65-75°F)

    par Conference A
        ZA->>HC: Set temperature {device: "hvac-conf-a", temp: 70}
        HC-->>ZA: Conf-A temperature command accepted
    and Conference B
        ZA->>HC: Set temperature {device: "hvac-conf-b", temp: 70}
        HC-->>ZA: Conf-B temperature command accepted
    and Conference C
        ZA->>HC: Set temperature {device: "hvac-conf-c", temp: 70}
        HC-->>ZA: Conf-C temperature command accepted
    end

    ZA->>AUD: Log bulk temperature change
    AUD-->>ZA: Audit record created
    ZA-->>WD: Success: All zones updated to 70°F
    WD-->>FM: Display confirmation with zone status

    Note over FM,AUD: Authorization failure case
    FM->>WD: Attempt to control restricted zone
    WD->>ZA: POST /api/zones/temperature {zone: "Server-Room", temp: 65}
    ZA->>AS: Verify user permissions for Server-Room
    AS-->>ZA: Access denied - insufficient privileges
    ZA-->>WD: HTTP 403: Unauthorized for zone Server-Room
    WD-->>FM: Error: Access denied for Server-Room
```

#### HVAC System Event Flow Diagram

```mermaid
graph TB
    subgraph Data Sources
        TS[Temperature Sensors]
        HS[Humidity Sensors]
        AQ[Air Quality Sensors]
        OS[Occupancy Sensors]
        WX[Weather API]
    end

    subgraph Ingestion Layer
        SDI[Sensor Data Ingestion]
    end

    subgraph Event Platform
        ESP[Event Streaming Platform]
    end

    subgraph Processing Engines
        RTA[Real-time Analytics]
        HDP[Historical Data Processor]
        MLP[ML Prediction Service]
    end

    subgraph Response Systems
        HCS[HVAC Control Service]
        VCS[Ventilation Control Service]
        AMS[Alert Management Service]
        EOS[Energy Optimization Service]
        BMD[Building Management Dashboard]
    end

    subgraph Storage
        TSDB[(Time Series Database)]
        ML_Store[(ML Model Store)]
    end

    %% Data ingestion flows
    TS --> |temp.reading| SDI
    HS --> |humidity.reading| SDI
    AQ --> |air_quality.reading| SDI
    OS --> |occupancy.detected| SDI
    WX --> |weather.updated| SDI

    SDI --> ESP

    %% Real-time processing
    ESP --> |sensor.*| RTA
    ESP --> |weather.*| RTA

    %% Historical processing
    ESP --> |sensor.*| HDP
    ESP --> |sensor.*| TSDB

    %% ML processing
    HDP --> |patterns.identified| MLP
    TSDB --> MLP
    MLP --> ML_Store

    %% Response flows
    RTA --> |climate.adjustment_needed| HCS
    RTA --> |ventilation.adjustment_needed| VCS
    RTA --> |threshold.exceeded| AMS
    RTA --> |live.metrics| BMD

    MLP --> |prediction.climate_demand| ESP
    MLP --> |prediction.energy_usage| EOS

    %% Secondary event flows
    HCS --> |hvac.adjusted| ESP
    VCS --> |ventilation.adjusted| ESP
    AMS --> |alert.triggered| ESP
    EOS --> |schedule.optimized| ESP

    %% Dashboard updates
    ESP --> |hvac.adjusted| BMD
    ESP --> |alert.triggered| BMD
    ESP --> |schedule.optimized| BMD
```

#### HVAC System Architecture Diagram

```mermaid
graph TB
    subgraph External
        WS[Weather API]
        EP[Energy Pricing]
        ES[Emergency Services]
        CORP[Corporate Systems]
    end

    subgraph Devices
        SENS[Environmental Sensors]
        SEC[Security Devices]
        HVAC[HVAC/Lighting]
        ENERGY[Energy Systems]
    end

    subgraph Edge
        GW[Edge Gateway]
        TRANS[Protocol Translation]
        BUFF[Local Buffer]
    end

    subgraph Cloud
        INGEST[Device Ingestion Service]
        BUS[Event Bus]

        subgraph Services
            API[API Gateway]
            AUTH[Auth Service]
            HVAC_SVC[Central Control Service]
            SEC_SVC[Security Service]
            ANALYTICS[Analytics/ML]
        end

        subgraph Storage
            TSDB[(Time Series DB)]
            RDB[(Relational DB)]
            CACHE[(Cache)]
        end
    end

    subgraph Apps
        OPS[Operations Dashboard]
        MOBILE[Mobile App]
        EXEC[Executive Portal]
        TENANT[Tenant Portal]
    end

    %% Device to Edge
    SENS --> GW
    SEC --> GW
    HVAC --> GW
    ENERGY --> GW

    %% Edge Processing
    GW --> TRANS
    TRANS --> BUFF
    BUFF --> INGEST

    %% External Data
    WS --> INGEST
    EP --> INGEST

    %% Data Flow to Storage
    INGEST -->|Sensor readings| BUS
    BUS -->|Readings to validate and transform| HVAC_SVC
    BUS -->|Sensor events| ANALYTICS
    HVAC_SVC -->|Control events| BUS
    BUS -->|Device commands| GW
    GW --> HVAC

    %% App Layer
    OPS --> API
    MOBILE --> API
    EXEC --> API
    TENANT --> API

    %% API routing
    API --> AUTH
    API --> HVAC_SVC
    API --> SEC_SVC
    API --> ANALYTICS

    %% Service data access
    HVAC_SVC -->|Processed sensor history| TSDB
    AUTH -->|User accounts and permissions| RDB
    SEC_SVC --> RDB
    ANALYTICS --> TSDB
    HVAC_SVC --> CACHE

    %% External integrations
    SEC_SVC --> ES
    API --> CORP

    %% Analytics outputs
    ANALYTICS --> EXEC
```

## Microservices and Modular Design Patterns

### From Systems to Services

- **Choosing between a monolith and separate services involves trade-offs.** Evaluate development speed, deployment, scaling, and operational complexity against business needs.
- **Monolith:** Packages functionality into one application deployed as a unit.
  - In the example, user authentication, product catalog, shopping cart, and payment processing share a database.
  - Supports fast development, straightforward deployment, and debugging within one system.
  - Updating payment processing requires redeploying the whole application.
  - Scaling for heavy product-search traffic also scales other functionality, such as authentication, even when it does not need extra capacity.
- **Services:** Separate specialized responsibilities so they can be deployed and scaled individually.
  - Increase the number of systems and deployments to manage; splitting one application into five services means operating five deployments.
  - Replace internal method calls across the extracted boundaries with network calls that can fail, time out, or respond slowly.
- **Selective extraction is a valid middle ground.** Keep most functionality together when it has similar needs and separate only the part that benefits from independence.
  - Example: Extract payment processing while authentication, the catalog, and the cart remain in the monolith.
- **Choose according to team size, operational capabilities, scaling needs, and tolerance for complexity.** A practical architecture that meets business requirements is more valuable than pursuing technical perfection.

![Microservices Idea Tradeoff](./assets/microservices_idea_tradeoff.png)

![Microservices Alternative](./assets/microservices_alternative.png)

### Form Systems to Services: Exercise

- **Task:** As SimpleShop's lead architect, identify the limitations of its growing e-commerce monolith and propose a service-oriented structure.
  - The original codebase handles accounts, login, and profiles; product browsing, inventory, and search; cart updates; checkout, taxes, shipping, and payments; confirmation emails and shipping notifications.
- **Problems to address:**
  - **Development speed:** Tight coupling allows small changes to break unrelated functionality, creating fear of changes and slowing feature delivery.
  - **Deployment risk:** Every update redeploys the whole application; the scenario's deployment process also requires downtime.
  - **Database contention:** User, product, and order functions compete for one shared database during peak shopping periods.
  - **Scaling inefficiency:** Extra product-catalog traffic requires scaling the whole application, wasting resources on less-busy functions.
- **Proposed solution: Five focused services, each owning its data, logic, and reasons to change.**
  - **User Service:** Accounts, authentication, and profiles.
  - **Product Service:** Catalog, inventory, and search.
  - **Cart Service:** Adding, removing, and updating cart items.
  - **Order Service:** Checkout coordination and payment processing through a payment provider.
  - **Notification Service:** Order confirmations and shipping updates.
  - An API (Application Programming Interface) gateway provides client access, and an event bus carries background messages.
- **Checkout combines two communication patterns.**
  - **Synchronous calls provide immediate answers:** The user submits an order through the gateway; the Order Service retrieves cart contents, validates inventory with the Product Service, and processes payment.
    - The frontend can make its request asynchronously to remain responsive while the backend follows this request-response flow.
  - **Asynchronous events trigger background work:** After payment, the Order Service publishes `OrderCreated`; the Product Service updates inventory counts, and the Notification Service sends a confirmation email when it consumes the event.
  - **Image variation:** The checkout image shows a separate Payment Service, whereas the five-service proposal keeps payment responsibility in the Order Service.
- **Benefits and trade-offs:** Independent boundaries let the team scale the Product Service for Black Friday or update payment handling without redeploying unrelated services.
  - Distributed coordination adds complexity; tightly connected synchronous operations may be better kept within one larger service.
  - Extract only the responsibilities whose independence justifies the added operational cost.

![Microservices Exercise](./assets/microservices_exercises.png)

![Microservices Exercise Checkout Flow](./assets/microservoces_exercise_checkout_flow.png)

[Single Responsibility Principle](http://objectmentor.com/resources/articles/srp.pdf)

### Microservices: The Good, The Bad, and The Ugly

- **The good: Independent services can support technical and organizational growth.**
  - **Deployment:** Fix and redeploy a payment service without redeploying user management or the product catalog.
  - **Scaling:** Allocate additional capacity to the services that need it.
  - **Team autonomy:** Teams own services end to end and choose technologies suited to their responsibilities.
    - Examples: Python with TensorFlow for recommendations, or Go for performance-sensitive order processing, without requiring one company-wide language or framework.
  - **Fault isolation:** Well-designed boundaries can contain a service failure so it does not bring down the whole system.
- **The bad: Distribution adds operational complexity.**
  - Network calls across service boundaries introduce latency, timeouts, and partial failures in place of local method calls.
    - Teams need monitoring, logging, distributed tracing, circuit breakers, and retries, along with the skills to operate them.
  - Debugging an unsuccessful order can require correlating behavior across user, inventory, payment, and notification services.
    - Consistent logging helps reconstruct failures that span several systems.
  - Deployment coordination and testing across services become harder; running the entire system locally can become impractical.
  - Distributed data often needs eventual-consistency patterns instead of one shared ACID (Atomicity, Consistency, Isolation, Durability) transaction across the workflow.
- **The ugly: Costs can outweigh benefits when adoption is premature or ownership is misaligned.**
  - Three developers maintaining five services may gain little independence while taking on substantial distributed-system overhead.
  - One team responsible for every service captures fewer organizational benefits than teams with clear service ownership.
  - Simple applications and organizations without sufficient operational maturity are poor candidates for unnecessary decomposition.
- **Choose based on current constraints and demonstrated needs.**
  - Evaluate whether team capacity and DevOps (Development and Operations) practices can support multiple independently operated services.
  - Look for concrete needs for the deployment, scaling, and technology autonomy described above.
  - For many startups and small teams, a well-built monolith is more practical; extract services later when the benefits justify the costs.
  - Architecture should help the business ship today rather than imitate a much larger company's anticipated needs.

![Microservices Pros and Cons](./assets/microservices_pros_cons.png)

### Exercise: Monoliths vs. Microservices

- **Scenario:** CampusEvent is a newly funded startup helping university students discover and register for campus events.
  - **Features:** Clubs and departments create, edit, and publish listings; student registrations are tracked; organizers view attendance and other metrics.
  - **Differentiator:** Recommendations based on student interests and past registrations.
  - **Future capability:** Ticket purchases for paid events.
  - **Constraints:** Three developers, limited distributed-systems operational experience, and pressure to deliver an MVP (Minimum Viable Product) quickly to validate the concept.
- **Task:** Justify both an initial monolith and microservices from the start, considering current constraints and the long-term independence of recommendations and payments.
- **Case for a monolith:**
  - The small team can collaborate closely without needing separate service ownership.
  - Local function calls simplify development, debugging through a single stack trace, and testing without distributed test environments or mocked service interactions.
  - One deployment and runtime reduce the work required to build, host, expose, connect, and observe the application.
    - The team can instrument a system it understands holistically rather than spread its knowledge across several services.
- **Case for microservices:**
  - **Recommendation Engine:** Iterate, deploy, and scale independently, reducing the impact of changes on other features.
    - Could later be offered as a standalone API (Application Programming Interface) product.
  - **Future payment or ticket-sales service:** Give new developers focused ownership without requiring them to learn the entire application.
  - On-demand compute models, such as FaaS (Function as a Service), may offer cost benefits as suitable workloads scale.
- **Solution: Start with a monolith.**
  - The unproven product needs market validation and delivery speed more urgently than the flexibility described above.
  - Service boundaries are still uncertain: recommendations, payments, and users are candidate divisions that actual usage and domain knowledge must validate.
  - Distributed infrastructure setup and ongoing operations could exceed the team's capacity and delay shipping.
  - Starting with microservices can be justified by well-understood domains, validated use cases, and strong operational expertise; those conditions are absent here.
  - Revisit the decision as the business, product, and team mature; extract services when their benefits justify the costs.

### How Do I Make My Service Micro?

- **Migrate incrementally once team capacity, operational maturity, and concrete needs justify microservices.** Keep the business running while replacing selected parts of the monolith.
- **Use the Strangler Fig pattern to extract one capability at a time.**
  - Choose isolated, well-understood functionality whose failure has limited business impact, such as notifications, report generation, or noncritical audit logging.
  - Build it as a separate service, then route the relevant requests or API (Application Programming Interface) paths to it.
  - Feature flags can control whether the new implementation receives requests.
  - Repeat as appropriate; the original system can eventually be retired if its responsibilities are fully replaced.
  - The payment-extraction image illustrates the mechanism; business-critical payments need not be the safest first candidate.
- **Size services around cohesive business capabilities rather than lines of code or technical layers.**
  - Keep related logic and its data together instead of separating API, business logic, and data access into different services.
  - Aim for high cohesion within services and low coupling between them, allowing independent deployment without coordinated releases.
  - A service should be understandable and maintainable by its team while delivering meaningful business value.
- **Use DDD (Domain-Driven Design) to find natural boundaries and divide shared data ownership.**
  - Identify bounded contexts, within which domain terms and business rules have specific meanings.
  - Illustrative ownership:
    - **User Service:** Profiles, authentication tokens, and preferences.
    - **Inventory Service:** Product information, stock levels, and supplier data.
    - **Order Service:** Order history, shipping addresses, and payment records.
  - Avoid boundaries that require extensive database queries across services.
  - Expect to refine early decisions as domain understanding improves; the image moves order history from the User database to the Order database.
  - Avoid premature fragmentation: splitting later is often easier than repairing poorly chosen boundaries.
- **Plan compatibility and rollback while old and new implementations coexist.**
  - Use a canary deployment to expose the new service to a small percentage of users, monitoring errors and performance before expanding or reverting.
  - Simple weighted load balancing can provide gradual traffic routing when a full canary setup is unavailable.
  - Example progression: 5%, 20%, 50%, then 100% of relevant traffic once behavior and capacity are satisfactory.
  - Learn from each extraction and improve operational capabilities along the way; rushing can produce a distributed system harder to manage than the original monolith.

![Monolith to Microservices](./assets/monolith_to_microservice.png)

![Monolith to Microservices Step by Step](./assets/monolith_to_microservices_step.png)

![Domain-Driven Design (DDD): Database Partitions](./assets/ddd_database_partitions.png)

![Service Boundaries](./assets/service_boundaries.png)

![Gradually Route Traffic](./assets/gradually_route_traffic.png)

Additional material:

- [Strangler Fig Pattern - Martin Fowler](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Domain Driven Design - Microsoft](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)

### Communicating Microservices

- **Microservices distribute decision-making as well as code, making human coordination essential.**
  - An authentication change that one monolith team could implement, test, and deploy may require agreement across user, order, notification, and recommendation teams.
- **Conway's Law:** System architecture tends to reflect the organization's communication structure.
  - Teams that rarely communicate risk producing services that integrate poorly.
  - One team managing many services may take coordination shortcuts that introduce tight coupling.
  - Treat organizational structure and service design as connected concerns; technical patterns cannot compensate for ineffective collaboration.
- **Adapt explanations to the audience.**
  - **Executives:** Connect architectural decisions to delivery speed, team scalability, and operational costs.
  - **Developers:** Explain service boundaries, API (Application Programming Interface) contracts, and integration patterns.
    - Record why boundaries exist so future developers can decide where new features belong, such as in a User Service or a Profile Service.
- **Maintain living documentation as the system changes.**
  - **Service maps:** Show dependencies and failure impact; the example maps the Order Service's dependencies on User, Product, Payment, and Notification services.
    - Answer questions such as what breaks if Payment fails and how user registration flows through the system.
    - Support onboarding, incident response, and capacity planning.
  - **Automatically tested API contracts:** Check that service interactions remain compatible.
  - **ADRs (Architectural Decision Records):** Preserve the reasoning behind design choices over time.
- **Use communication to assess the design.** Difficulty explaining a simple user action can signal poor service boundaries or excessive decomposition.
  - Dense dependency maps or a lengthy, multi-diagram explanation of registration are reasons to reconsider the design.
  - Share context continuously and discuss uncertain tasks together to keep decisions aligned across teams.

![Conway's Law](./assets/conways_law.png)

![Service Maps](./assets/service_maps.png)

Further reading: [Conway's Law](https://martinfowler.com/bliki/ConwaysLaw.html)

### Exercise: Moving to Microservices

- **Scenario:** QuickBite, a fast-casual restaurant chain, runs a legacy monolith covering menu management, cashier and kiosk orders, kitchen displays, loyalty points and discounts, receipts, and credit-card payments.
  - Even a menu-price change requires full redeployment, creating risk and downtime during busy lunch periods.
  - Planned capabilities include mobile online ordering and a personalized loyalty program.
- **Task:** Design a phased migration that preserves existing functionality.
  - Choose the first extraction based on high value, low risk, and minimal dependencies.
  - Explain how the monolith retrieves current menu items and prices during the transition.
  - Propose at least three eventual services, defining each bounded context, responsibilities, and exclusively owned data.
- **Solution: Extract Menu Service first.**
  - Its responsibility and consumers are clear, complexity is relatively low, and routine restaurant use mainly reads its data.
  - It avoids starting with payment processing, whose failure could stop business operations.
  - Store menu items and prices in a separate, service-owned database so price changes become data updates rather than application redeployments.
  - Use the extraction to validate migration patterns and demonstrate business value before tackling more complex functions.
- **Apply the Strangler Fig pattern:** Replace monolith capabilities incrementally while keeping familiar interfaces available to existing callers.
  - Adapt the monolith's existing menu interface to delegate requests to the new service behind the scenes.
  - Other monolith components continue using that interface while receiving the latest menu data from the service.
- **Candidate service boundaries:**
  - **Menu Service:** Creates, updates, and displays menu items and prices, owning the menu database described above.
  - **Order Service:** Handles cashier and kiosk order creation; reuse suitable existing code and interfaces to preserve the user experience.
  - **Kitchen Service:** Coordinates preparation and kitchen displays by subscribing to order events.
  - **Loyalty Service:** Tracks points and applies discounts, primarily reacting to order events; some features may require additional interactions.
  - **Payment/Receipt Service:** Processes credit-card payments and generates receipts; reliability is critical to continued sales.
- **Add mobile ordering as a new service after validating the approach, potentially as the second or third extraction.**
  - Reuse the migration interfaces and patterns rather than adding this capability to the monolith being replaced.
  - The final diagram combines an Order Service, an event bus, kitchen and menu services, mobile ordering, and payment processing.

![QuickByte Problem](./assets/quickbyte_problem.png)

![QuickByte Services](./assets/quickbyte_services.png)

![QuickByte Microservices](./assets/quickbyte_microservices.png)

### Microservice Patterns: Resilience

- **Design for partial failure:** Dependencies can fail or slow down, so contain problems and preserve useful functionality rather than allowing cascading outages.
- **Circuit breaker:** Stop repeatedly calling a failing dependency, such as a payment service returning errors or timing out.
  - **Closed:** Allow calls and track failures; crossing the failure threshold opens the circuit.
  - **Open:** Fail fast rather than consuming resources while waiting; use a fallback when appropriate.
  - **Half-open:** After a cooldown, allow limited test calls; recovery closes the circuit, while further failures reopen it.
    - The image labels this state "Limited Closed."
- **Bulkhead:** Isolate resource pools or budgets for different operations so one workload cannot exhaust capacity needed by another.
  - Separate thread pools, connection pools, and memory allocations where isolation is needed.
  - Example: Slow report generation should not block critical authentication requests by consuming all shared resources.
- **Timeouts:** Limit waiting so stalled requests do not hold resources indefinitely.
  - Too short rejects legitimate slow responses; too long allows slow dependencies to degrade the wider system.
- **Retries:** Recover from transient failures without overwhelming a struggling service.
  - Too many attempts amplify load; too few abandon recoverable operations prematurely.
  - **Exponential backoff:** Increase the delay between successive attempts.
  - **Jitter:** Randomize those delays to spread retries over time, avoiding synchronized load spikes known as the thundering herd problem.
- **Meaningful health checks:** Assess whether a service can perform its role, beyond whether its web server returns an "OK" response.
  - Check database connectivity, dependency reachability, and queue backlogs.
  - Distinguish healthy, degraded-but-functional, and unavailable states; orchestration systems can route traffic away from instances unable to serve requests.
  - SRE (Site Reliability Engineering) practices and SLOs (Service Level Objectives) help assess service quality beyond a binary health status.
- **Graceful degradation:** Coordinate engineering and user-experience decisions so dependency failures reduce capabilities without breaking the whole experience.
  - Use cached data, defaults, or reduced feature sets when appropriate.
  - Replace unavailable personalized recommendations with popular products.
  - Omit slow reviews from product listings while retaining them on detail pages.

![Circuit Breaker Pattern](./assets/circuit_breaker_pattern.png)

![Bulkhead Pattern](./assets/bulkhead_pattern.png)

![Retry Patterns](./assets/retry_patterns.png)

![Health Checks](./assets/health_checks.png)

Additional material:

- [Circuit Breaker Pattern](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Site Reliability Engineering (or SRE)](https://sre.google/sre-book/introduction/)
- [SLOs](https://sre.google/sre-book/service-level-objectives/)

### Microservice Patterns: Consistency

- **Separate databases turn cross-service updates into a coordination problem.** A monolith can update related records, such as profiles and order history, within one database transaction; independently owned stores do not share that transaction automatically.
  - **ACID (Atomicity, Consistency, Isolation, Durability):** Transaction guarantees still apply within individual services; distributed transactions can extend coordination across stores at additional performance and complexity cost.
  - **BASE (Basically Available, Soft State, Eventual Consistency):** An alternative approach allows temporary differences while updates propagate.
    - Convergence requires reliable propagation and recovery logic; inconsistent data does not repair itself automatically.
    - Use idempotent operations so retries do not repeat business effects, and account for processing delays.
  - **CAP (Consistency, Availability, Partition Tolerance):** During a network partition, a distributed system cannot guarantee both linearizable consistency and availability for every request; this does not mean all microservices must abandon strong consistency.
- **Saga:** Replace one cross-service transaction with a sequence of local transactions and compensating actions for completed steps when later work fails.
  - Example: Reserve inventory, charge the credit card, then create a shipping record.
    - If payment fails, release the inventory reservation.
    - If shipping setup fails after payment, refund the charge and release inventory.
  - Compensation restores an acceptable business outcome; it may require multiple actions rather than literally reversing each change.
- **Event sourcing:** Store the sequence of state-changing events as the source of truth and reconstruct state by replaying them.
  - Example for order `1001`: `OrderPlaced`, `PaymentProcessed`, `ItemsShipped`, then `OrderDelivered` yields the current status `Delivered`.
  - Provides an audit trail and supports reconstructing historical states from the recorded events.
  - Requires event-centered state handling and managed event storage and delivery.
- **CQRS (Command Query Responsibility Segregation):** Separate models for state-changing commands and data-reading queries so each can be optimized independently.
  - Writes may require strong consistency, while read views may tolerate staleness for higher throughput.
  - Models can use different representations and scale independently; separate databases are optional ([CQRS reference](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)).
  - Social-media example: Posting writes to a transaction-optimized database; profile views read from a query-optimized replica that may lag by seconds.
- **Choose consistency guarantees according to business requirements.**
  - **User preferences:** Often tolerate delayed propagation.
  - **Shopping carts:** An item may become unavailable before checkout; revalidate stock there rather than locking inventory whenever someone adds it to a cart.
  - **Permissions:** AWS (Amazon Web Services) IAM (Identity and Access Management) changes can take time to become visible; clients must account for propagation delays ([AWS reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot.html)).
  - **Financial transactions:** Need strong correctness guarantees; represent pending transfer steps explicitly rather than assuming money can safely disappear or be duplicated temporarily.
  - **Safety-critical operations:** Need consistency and response-time guarantees appropriate to their safety requirements.
  - Design user interactions to tolerate delay where acceptable and reserve stronger coordination for invariants that require it.

![Data Consistency](./assets/data_consistency.png)

![ACID vs BASE](./assets/acid_vs_base.png)

![Saga Pattern: Idea](./assets/saga_pattern_idea.png)

![Saga Pattern: Example](./assets/saga_pattern_example.png)

![Event Sourcing: Example](./assets/event_sourcing_example.png)

![Event Sourcing: Advantages and Requirements](./assets/event_sourcing_advantages.png)

![CQRS Concept](./assets/cqrs_concept.png)

![CQRS Example: Social Media](./assets/cqrs_example_social_media.png)

Additional material:

- [Event Sourcing Pattern - Martin Fowler](https://martinfowler.com/eaaDev/EventSourcing.html)
- [CQRS Pattern - Martin Fowler](https://martinfowler.com/bliki/CQRS.html)
- [ACID Principles](https://www.databricks.com/glossary/acid-transactions)
- [BASE Principles](https://devonblog.com/software-development/base-principles/)
- [CAP Theorem](https://robertgreiner.com/cap-theorem-revisited/)

### Microservice Patterns: Performance


### Microservice Patterns: Exercise


## Cloud-Native Architectural Patterns

## Data and AI Architecture Patterns

## Distributed Architecture Patterns

## Project: Real-Time Financial Fraud Detection -- Solution Design

