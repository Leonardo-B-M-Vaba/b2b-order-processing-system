# System Architecture

This project follows a **Modular Monolith architecture** combined with Domain Driven Design principles.

The application is divided into independent modules representing business capabilities.

## Modules

- Order Decision Engine (Core Domain)
- Orders
- Shipment
- Invoices

## Architecture Layers

API → Application → Domain → Infrastructure

## Communication

Modules communicate using:

- Application services
- Domain events
