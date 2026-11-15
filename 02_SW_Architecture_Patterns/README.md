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
  - [Microservices and Modular Design Patterns](#microservices-and-modular-design-patterns)
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
  - Routes requests to the appropriate backend service, such as orders or restaurants, and can centralize authentication, logging, and rate limiting.
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

## Microservices and Modular Design Patterns

## Cloud-Native Architectural Patterns

## Data and AI Architecture Patterns

## Distributed Architecture Patterns

## Project: Real-Time Financial Fraud Detection -- Solution Design

