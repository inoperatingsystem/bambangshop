# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In my understanding, using a trait for Subscriber in Rust is not strictly required for the current BambangShop scope, because the app currently has one concrete observer behavior: send notification to one HTTP endpoint URL. A single struct is enough to represent subscriber data and can still follow Observer principles as long as the publisher only depends on the observer contract conceptually (subscribe, unsubscribe, notify). However, a trait becomes valuable when we expect multiple observer implementations in the future, for example webhook subscriber, email subscriber, logging subscriber, or mock subscriber for testing. So for this tutorial stage, a single model struct is practical and simple, while introducing a trait is more about extensibility and cleaner abstraction for future growth.

2. For uniqueness of id in Program (or Product in this codebase) and url in Subscriber, Vec is functionally possible but less efficient and more error-prone because we must manually scan the list to check duplicates, find items, and delete items in O(n) time. A map structure is more aligned with uniqueness requirements because key uniqueness is built into the data model and lookup/delete become near O(1). That is why using a map-like structure is a better fit when uniqueness is a core rule. In short, Vec is sufficient only for very small data and simple learning scenarios, but DashMap or HashMap is the better structural choice when we treat id and url as unique identifiers.

3. Rust’s compiler guarantees memory safety and prevents many data-race bugs, but it does not automatically choose the best shared-state structure for us. Singleton and DashMap solve different problems: Singleton controls instance count, while DashMap provides thread-safe concurrent access. In this project, the static subscribers storage is already singleton-like because it is globally initialized once, but we still need synchronization for concurrent requests from Rocket handlers. So Singleton alone is not enough; we would still need a lock-based or concurrent container inside it. This means keeping DashMap (or alternatively Mutex/RwLock around HashMap) is still necessary for thread-safe shared mutation, and the current choice is reasonable for concurrent Observer registration and notification workflows.

#### Reflection Publisher-2
1. In my understanding, separating Service and Repository from Model is mainly about applying separation of concerns and keeping each module focused on one responsibility. Repository focuses on data access concerns such as storing, retrieving, and deleting entities, while Service focuses on business rules and application flow such as validation, orchestration, and error handling. If all of this is merged into the Model, one struct or module ends up carrying too many reasons to change: changes in persistence mechanism, changes in API behavior, and changes in business policy. By splitting them, the code becomes easier to test, easier to maintain, and easier to extend. This is also aligned with clean architecture ideas, where domain data, business logic, and infrastructure concerns are not tightly coupled.

2. If we only use Model for everything, the interactions between Program, Subscriber, and Notification would likely become tightly coupled and harder to reason about. For example, Program model might directly know how to store subscribers, how to build notifications, and how to send external requests; Subscriber model might also contain persistence details and transport logic; Notification model might become a mixed object that both represents message data and controls delivery flow. Over time, each model would grow into a “god object” with many methods and side effects, making it difficult to isolate bugs, reuse logic, or write focused unit tests. I imagine small feature requests, such as changing subscription uniqueness rules or notification retry strategy, would force edits across multiple large models and increase regression risk.

3. I have explored Postman more, and it is very helpful for validating REST API work quickly and consistently. For my current progress, Postman helps me test endpoint behavior end-to-end, including status codes, request body formats, query parameters, and response payloads without writing a frontend first. The most useful features for me are Collections (to organize API scenarios), Environments (to switch base URLs and variables easily), Tests scripts (to automate response assertions), and Runner (to run many requests as a workflow). I am also interested in pre-request scripts for token generation and mock servers for parallel development. For group projects and future software engineering work, these features help standardize API testing, reduce manual mistakes, and improve collaboration because teammates can share the same executable API test documentation.

#### Reflection Publisher-3
1. In this tutorial, we use the Push variation of the Observer Pattern. The publisher side (our BambangShop app) actively sends notification data to each subscriber endpoint when an event happens, such as product creation or deletion. Subscriber instances do not request updates manually; instead, they receive the payload directly from the publisher. This is why the flow feels event-driven: once an event is triggered in the publisher, information is immediately pushed outward to all registered observers of that product type.

2. If we imagine switching to Pull, the main advantage is simpler publisher responsibility because it only needs to signal that an update exists, while subscribers decide when and what to fetch. This can reduce payload coupling and let subscribers request only the data they need. However, for this tutorial case, Pull also brings disadvantages: subscribers need extra logic to poll or request details, there are more HTTP calls overall, and update timeliness depends on subscriber polling strategy. In practice, Push is more straightforward for near real-time notification tutorials, while Pull can be better when data is large, subscriber needs differ significantly, or clients must control fetch timing.

3. If we do not use multi-threading in the notification process, notification delivery becomes sequential, so the program will send to subscribers one by one. The direct effect is slower response time for publish-related operations, especially when some subscriber endpoints are slow or temporarily unresponsive. A single slow subscriber can delay all following notifications, reducing throughput and making the system feel less responsive under load. Functionally the program can still work, but scalability and fault tolerance become weaker because the notification pipeline is more easily blocked by network latency or endpoint issues.
