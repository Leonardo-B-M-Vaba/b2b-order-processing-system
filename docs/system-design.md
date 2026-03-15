# System Context

Client Applications
      │
      ▼
   API Layer
      │
      ▼
 Application Services
      │
      ▼
 Domain Layer
      │
      ▼
 Infrastructure

 # Key Architectural Decisions

* Modular Monolith chosen to balance simplicity and modularity.
* DDD used to model complex business rules.
* Event-driven communication used between modules.

# Module Boundaries 

- Orders
- Decision Engine
- Shipment
- Invoices
