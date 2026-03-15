# Domain Model

The system models the business processes involved in B2B order processing. This document outlines the core entities, their relationships, and the aggregates that structure the application.

## Main Entities

### Order

Represents a purchase order submitted by a customer to request products or services.

**Attributes:**
- `Id` - Unique identifier for the order
- `CustomerId` - Reference to the customer placing the order
- `OrderItems` - Collection of items (SKUs and quantities) included in the order
- `TotalAmount` - Calculated total cost of all items in the order
- `Status` - Current state of the order (e.g., Pending, Confirmed, Shipped, Delivered, Cancelled)

**Relationships:**
- Linked to a `Contract` to validate credit limits and terms
- Contains references to `InventoryItem` entities for stock validation
- Generates a `Shipment` once confirmed
- Produces an `Invoice` for payment processing

### Contract

Represents the commercial agreement between the company and a customer, defining terms and credit arrangements.

**Attributes:**
- `ContractId` - Unique identifier for the contract
- `CustomerId` - Reference to the customer
- `CreditLimit` - Maximum allowed outstanding balance for the customer
- `ExpirationDate` - When the contract terms expire and require renewal

**Relationships:**
- Referenced during order validation to enforce credit limits
- Multiple orders may reference the same contract

### InventoryItem

Represents the available stock for a specific product in the warehouse.

**Attributes:**
- `ProductId` - Unique identifier for the product
- `AvailableQuantity` - Current quantity available for purchase

**Relationships:**
- Checked during order placement to validate stock availability
- Updated when orders are confirmed or shipments are dispatched

### Shipment

Represents the dispatch of ordered items to a customer.

**Attributes:**
- `ShipmentId` - Unique identifier for the shipment
- `OrderId` - Reference to the associated order
- `Items` - List of items being shipped (may be partial if backordered)
- `TrackingNumber` - Carrier tracking identifier
- `DispatchDate` - When the shipment leaves the warehouse
- `Status` - Current state (e.g., Pending, In-Transit, Delivered)

**Relationships:**
- Created from an `Order` once items are reserved from inventory
- References inventory to decrement available quantities

### Invoice

Represents the billing document for an order, used for payment tracking and accounting.

**Attributes:**
- `InvoiceId` - Unique identifier for the invoice
- `OrderId` - Reference to the associated order
- `Amount` - Total amount due
- `IssuedDate` - When the invoice was created
- `DueDate` - Payment deadline
- `Status` - Current state (e.g., Issued, Partially Paid, Paid, Overdue)

**Relationships:**
- Created from an `Order` for billing purposes
- Tracks payment against the customer's `Contract` credit limit

---

## Aggregates

Aggregates are cohesive groups of entities that maintain consistency boundaries:

1. **Order Aggregate** - Encompasses `Order` and `OrderItems`, enforcing business rules around order creation and modification
2. **Contract Aggregate** - Encompasses `Contract` and credit validation rules, ensuring credit limits are respected
3. **Inventory Aggregate** - Encompasses `InventoryItem` and stock management, maintaining accurate available quantities
4. **Shipment Aggregate** - Encompasses `Shipment` and dispatch details, managing fulfillment workflows
5. **Invoice Aggregate** - Encompasses `Invoice` and payment tracking, managing billing and accounts receivable

---

## Key Business Rules

- Orders cannot exceed a customer's available credit (from their `Contract`)
- Orders cannot be placed for quantities exceeding available `InventoryItem` quantities
- `Shipments` are generated only after order confirmation and inventory reservation
- `Invoices` are generated upon order fulfillment and must be satisfied before contract renewal