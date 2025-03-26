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
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
>In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?

In this case, a single Model struct is sufficient rather than using a trait (Rust's version of an interface) because the current implementation of BambangShop only deals with one type of subscriber. Since there aren't multiple subscriber implementations with different behaviors that need to conform to a common interface, there's no need for the polymorphism that a trait would provide. If we were to implement different types of subscribers with different behaviors within the same application, then a trait would be more appropriate to enforce the common interface while allowing different implementations.

>id in Program and url in Subscriber is intended to be unique. Explain based on your understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?

Using DashMap (map/dictionary) rather than Vec (list) is necessary in this case because both Program id and Subscriber url need to be unique identifiers that allow for efficient lookups. When using a Vec, finding or removing a specific subscriber would require iterating through the entire collection with O(n) time complexity, which becomes inefficient as the number of subscribers grows. DashMap provides O(1) lookup, insertion, and deletion operations based on the key (url), making operations much faster. Additionally, DashMap is thread-safe by design, which is crucial for a web application where multiple users might be subscribing or unsubscribing concurrently. The nested structure (DashMap within DashMap) also efficiently organizes subscribers by product type, allowing quick access to all subscribers of a particular product type without scanning the entire collection.

>When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we used the DashMap external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need DashMap or we can implement Singleton pattern instead?

 In the case of the List of Subscribers (`SUBSCRIBERS`) static variable, we used the DashMap external library for thread-safe HashMap implementation. In the current implementation, we're already using a form of the Singleton pattern through the `lazy_static` macro, which ensures `SUBSCRIBERS` is initialized only once when first accessed. What DashMap provides is thread-safe concurrent access with fine-grained locking, which is essential in web application where multiple threads might modify the subscribers list simultaneously. If we tried to implement our own thread-safe HashMap with a traditional Singleton approach, we would have to handle the synchronization manually using Mutex or RwLock, which would be more complex and potentially less performant than DashMap's specialized implementation.

#### Reflection Publisher-2

>In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?

Separating "Service" and "Repository" from a "Model" addresses the Single Responsibility Principle (SRP) more effectively than traditional MVC. While MVC combines data storage and business logic in the Model component, this often leads to bloated Model classes that become difficult to maintain as applications grow. By extracting the Repository layer, we create a dedicated component focused solely on data persistence concerns, making database interactions more manageable and testable in isolation. Similarly, moving business logic to a Service layer allows us to encapsulate complex operations and workflows separately from data structures. This separation creates clearer boundaries between components, improves testability through easier mocking, and enables better reuse across different parts of the application. It also facilitates maintenance by allowing changes to one layer (like switching database technologies) without affecting others.

>What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?

If we only used the Model without separating Service and Repository layers, the complexity of each model would increase dramatically. For instance, our Program model would need to handle not just its own data structure but also business logic for product management, subscriber notifications, and persistence logic for database operations. This would create tight coupling between models, as Program would need direct knowledge of Subscriber and Notification models to manage their relationships. Similarly, Subscriber would need to know how to store itself and also how to interact with Notification objects. Each model would become a massive class with multiple responsibilities, violating the Single Responsibility Principle. When we needed to modify notification logic, we would need to dive into the Subscriber model, risking unintended side effects on subscription data handling. Testing would become more challenging as we couldn't isolate business logic from data persistence concerns. Additionally, as our application grew, these models would continue to expand with more methods and properties, making them increasingly difficult to understand and maintain. The lack of clear boundaries would create a web of interdependencies that would make future changes risky and time-consuming.

>Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.

Postman has been incredibly helpful for testing our BambangShop notification system. It allows us to easily send HTTP requests to both our publisher and subscriber applications without having to build a frontend interface. We can quickly create, save, and organize different API requests for subscribing to product types, creating products, and triggering notifications. The collection feature has been particularly useful as we can group related requests together and run them in sequence to test the entire notification flow.

I'm especially interested in Postman's environment variables feature, which lets us switch between testing against different server instances (like localhost:8000 for the main app and localhost:8001 for the receiver app). The request and response visualization helps us quickly identify issues in our API implementation.

#### Reflection Publisher-3
