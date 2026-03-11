# Domain Model

The system models the business processes involved in B2B order processing.

## Main Entities

### Order

Represents a purchase order submitted by a company.

Attributes:

- Id
- CustomerId
- OrderItems
- TotalAmount
- Status

### Contract

Represents the commercial agreement between the company and the supplier.

Attributes:

- ContractId
- CustomerId
- CreditLimit
- ExpirationDate

### InventoryItem

Represents the available stock for a product.

Attributes:

- ProductId
- AvailableQuantity
