# Order Decision Engine

The Order Decision Engine is responsible for evaluating incoming orders and determining how they should be processed.

## Validation Steps

1. Contract Validation
2. Credit Limit Validation
3. Inventory Check
4. Approval Rules

## Possible Outcomes

- APPROVED
- REJECTED
- MANUAL_REVIEW
- APPROVAL_REQUIRED

## Workflow

Order Received  
↓  
Validate Contract  
↓  
Check Credit Limit  
↓  
Check Inventory  
↓  
Apply Approval Rules  
↓  
Decision
