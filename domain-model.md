# Domain Model

The system models the business processes involved in B2B order processing using Domain-Driven Design principles.

## Overview

This domain model describes the core entities, value objects, and aggregates that form the business domain of the B2B Order Processing System.

---

## Value Objects

Value Objects represent immutable, attribute-based concepts without identity. They are defined by their attributes and are not tracked by identity.

### OrderStatus
Represents the current state of an order in the processing workflow.

Possible values:
- `Pending` - Order received, awaiting decision engine evaluation
- `Approved` - Order approved for processing
- `Rejected` - Order rejected by business rules
- `ManualReview` - Order requires human review
- `ApprovalRequired` - Order requires manager approval

### Money / OrderAmount
Represents a monetary value associated with order totals.

Attributes:
- Currency (default: USD)
- Amount (decimal)

### CustomerId
Represents the unique identifier of a customer. Encapsulates customer identity.

### ContractId
Represents the unique identifier of a contract.

### ProductId
Represents the unique identifier of a product.

---

## Aggregates

An Aggregate is a cluster of associated objects that are treated as a single unit for the purpose of data change. Each aggregate has a root entity that is the only object of the aggregate that outside objects are allowed to hold references to.

### 1. Order Aggregate

**Aggregate Root:** Order

**Entities:**
- Order (root)
- OrderItem (child entity)

**Value Objects:**
- OrderStatus
- OrderAmount
- CustomerId

**Description:**
The Order aggregate represents a purchase order submitted by a customer. It encapsulates all information related to a single order and its items.

**Attributes:**

#### Order (Root Entity)
- `OrderId` (Identity) - Unique identifier for the order
- `CustomerId` (Value Object) - Reference to the customer who placed the order
- `OrderItems` (Collection of OrderItem) - Line items in the order
- `TotalAmount` (Money) - Total cost of the order
- `Status` (OrderStatus) - Current state of the order
- `CreatedAt` (DateTime) - Order creation timestamp
- `UpdatedAt` (DateTime) - Last update timestamp

#### OrderItem (Child Entity)
- `OrderItemId` (Identity) - Unique identifier within the aggregate
- `ProductId` (Value Object) - Reference to the product
- `Quantity` (int) - Number of units ordered
- `UnitPrice` (Money) - Price per unit
- `LineTotal` (Money) - Quantity × UnitPrice

**Key Behaviors:**
- Create a new order with items
- Add/remove order items
- Calculate total amount from items
- Update order status
- Publish OrderCreatedEvent when created
- Publish OrderStatusChangedEvent when status changes

**Invariants (Business Rules):**
- Order must contain at least one item
- Total amount must equal sum of all line totals
- Only specific status transitions are allowed
- Status can only be changed by the Order Decision Engine

---

### 2. Contract Aggregate

**Aggregate Root:** Contract

**Value Objects:**
- ContractId
- CustomerId
- ExpirationDate
- CreditLimit (Money)

**Description:**
The Contract aggregate represents a commercial agreement between the company and a customer. It defines the terms under which orders from that customer can be accepted.

**Attributes:**

#### Contract (Root Entity)
- `ContractId` (Identity) - Unique identifier for the contract
- `CustomerId` (Value Object) - Which customer this contract is with
- `CreditLimit` (Money) - Maximum credit the customer can use
- `ExpirationDate` (DateTime) - When the contract expires
- `Status` (ContractStatus) - Active, Expired, Suspended, etc.
- `CreatedAt` (DateTime) - Contract creation date
- `UpdatedAt` (DateTime) - Last modification date

**Key Behaviors:**
- Verify if contract is active and valid
- Validate if contract matches a customer
- Check if credit limit is sufficient for an amount
- Publish ContractValidatedEvent

**Invariants (Business Rules):**
- Contract must have a valid CustomerId
- CreditLimit must be greater than zero
- ExpirationDate cannot be in the past for active contracts
- A customer can have multiple active contracts

---

### 3. Inventory Aggregate

**Aggregate Root:** InventoryItem

**Value Objects:**
- ProductId
- AvailableQuantity

**Description:**
The Inventory aggregate manages the available stock for products. It tracks how many units of each product are available for sale.

**Attributes:**

#### InventoryItem (Root Entity)
- `InventoryItemId` (Identity) - Unique identifier
- `ProductId` (Value Object) - Which product this inventory is for
- `AvailableQuantity` (int) - Number of units currently available
- `ReservedQuantity` (int) - Number of units reserved by approved orders
- `WarehouseLocation` (string) - Location in warehouse (optional)
- `LastUpdated` (DateTime) - Last time inventory was updated

**Key Behaviors:**
- Check if sufficient quantity is available
- Reserve inventory for an approved order
- Release reserved inventory if order is cancelled
- Update available quantity
- Publish InventoryReservedEvent
- Publish LowInventoryWarningEvent

**Invariants (Business Rules):**
- AvailableQuantity cannot be negative
- ReservedQuantity cannot exceed AvailableQuantity + ReservedQuantity
- ProductId must be valid
- Quantity changes must be tracked with timestamps

---

### 4. Shipment Aggregate

**Aggregate Root:** Shipment

**Entities:**
- Shipment (root)
- ShipmentItem (child entity)

**Value Objects:**
- ShipmentStatus
- CustomerId

**Description:**
The Shipment aggregate represents the fulfillment of an approved order. It tracks the packing and shipping of order items to the customer.

**Attributes:**

#### Shipment (Root Entity)
- `ShipmentId` (Identity) - Unique identifier
- `OrderId` (Reference) - Reference to the order being shipped
- `CustomerId` (Value Object) - Destination customer
- `ShipmentItems` (Collection) - Items being shipped
- `Status` (ShipmentStatus) - Pending, Shipped, Delivered, etc.
- `TrackingNumber` (string) - Carrier tracking number
- `ShippedDate` (DateTime) - When shipment left warehouse
- `ExpectedDeliveryDate` (DateTime) - Estimated delivery date
- `ActualDeliveryDate` (DateTime?) - When delivered (if applicable)

#### ShipmentItem (Child Entity)
- `ShipmentItemId` (Identity)
- `OrderItemId` (Reference) - Reference to original order item
- `QuantityShipped` (int) - How many units shipped
- `Condition` (ItemCondition) - Status of the item

**Key Behaviors:**
- Create shipment from approved order
- Update shipment status
- Publish ShipmentCreatedEvent
- Publish ShipmentDeliveredEvent

**Invariants (Business Rules):**
- Shipment must reference a valid approved order
- Cannot ship more items than ordered
- ShippedDate cannot be before OrderCreatedDate
- Status transitions must follow business rules

---

### 5. Invoice Aggregate

**Aggregate Root:** Invoice

**Entities:**
- Invoice (root)
- InvoiceLineItem (child entity)

**Value Objects:**
- InvoiceStatus
- CustomerId
- InvoiceNumber
- Money

**Description:**
The Invoice aggregate represents a bill for an approved and shipped order. It tracks payment terms and status.

**Attributes:**

#### Invoice (Root Entity)
- `InvoiceId` (Identity) - Unique identifier
- `InvoiceNumber` (Value Object) - Human-readable invoice number
- `OrderId` (Reference) - Reference to the order
- `CustomerId` (Value Object) - Billing customer
- `LineItems` (Collection of InvoiceLineItem) - Billable items
- `TotalAmount` (Money) - Total due
- `Status` (InvoiceStatus) - Draft, Issued, Paid, Overdue, etc.
- `IssuedDate` (DateTime) - When invoice was created
- `DueDate` (DateTime) - Payment due date
- `PaidDate` (DateTime?) - When paid (if applicable)

#### InvoiceLineItem (Child Entity)
- `LineItemId` (Identity)
- `OrderItemId` (Reference) - Reference to original order item
- `Description` (string)
- `Quantity` (int)
- `UnitPrice` (Money)
- `LineTotal` (Money)
- `TaxAmount` (Money) - Tax on this line item (optional)

**Key Behaviors:**
- Create invoice from shipped order
- Update payment status
- Calculate taxes
- Publish InvoiceCreatedEvent
- Publish InvoicePaidEvent

**Invariants (Business Rules):**
- Invoice must reference a valid shipped order
- TotalAmount must equal sum of line items plus taxes
- Cannot issue invoice for cancelled order
- DueDate must be after IssuedDate

---

## Aggregate Interactions and Cross-Aggregate References

**Important:** Aggregates communicate through their Root Entities by reference only. They do NOT directly share child entities.

### Order → Contract
When an order is received, the Order Decision Engine validates it against the Contract:
- Order knows CustomerId
- Engine looks up Contract by CustomerId
- Engine validates credit limit

### Order → Inventory
When validating an order:
- For each OrderItem, lookup InventoryItem by ProductId
- Check if AvailableQuantity is sufficient

### Order → Shipment
When an order is approved:
- A new Shipment aggregate is created with reference to OrderId
- Shipment copies relevant data but maintains separate identity

### Order → Invoice
When an order is shipped:
- A new Invoice aggregate is created with reference to OrderId
- Invoice copies item information but maintains separate identity

---

## Domain Events

Domain events are published by aggregates to notify other parts of the system of important business occurrences.

### Order Events
- `OrderCreatedEvent` - Triggered when a new order is created
- `OrderStatusChangedEvent` - Triggered when order status changes
- `OrderApprovedEvent` - Triggered when order is approved
- `OrderRejectedEvent` - Triggered when order is rejected
- `OrderRequiresManualReviewEvent` - Triggered when order needs human review

### Contract Events
- `ContractValidatedEvent` - Triggered when contract validation is successful
- `ContractValidationFailedEvent` - Triggered when contract is invalid

### Inventory Events
- `InventoryReservedEvent` - Triggered when inventory is reserved for an order
- `InventoryReleasedEvent` - Triggered when reserved inventory is released
- `LowInventoryWarningEvent` - Triggered when stock falls below threshold

### Shipment Events
- `ShipmentCreatedEvent` - Triggered when shipment is created
- `ShipmentShippedEvent` - Triggered when shipment leaves warehouse
- `ShipmentDeliveredEvent` - Triggered when shipment is delivered

### Invoice Events
- `InvoiceCreatedEvent` - Triggered when invoice is generated
- `InvoicePaidEvent` - Triggered when payment is received
- `InvoiceOverdueEvent` - Triggered when payment becomes overdue

---

## Summary Table

| Aggregate | Root Entity | Key Value Objects | Primary Responsibility |
|-----------|-------------|-------------------|------------------------|
| Order | Order | OrderStatus, Money, CustomerId | Capture and manage customer purchase requests |
| Contract | Contract | ContractId, CustomerId, Money | Define commercial terms with customers |
| Inventory | InventoryItem | ProductId, Quantity | Track available product stock |
| Shipment | Shipment | ShipmentStatus, CustomerId | Manage order fulfillment and delivery |
| Invoice | Invoice | InvoiceStatus, InvoiceNumber, Money | Track billing and payments |