# Data Model: Expense Tracker Application

## Overview

This document defines the data model and entity relationships for the expense tracker application. The model is designed to support efficient storage, retrieval, and analysis of financial transactions while maintaining data integrity and performance.

## Core Entities

### Expense

Represents a single financial transaction recorded by the user.

```typescript
interface Expense {
  id: string;                    // UUID - Primary identifier
  amount: number;                // Monetary amount in smallest currency unit
  date: Date;                    // Transaction date
  categoryId: string;            // Foreign key to Category
  paymentMethod: PaymentMethod;  // How the expense was paid
  description: string;           // Short description of the expense
  notes?: string;                // Optional additional notes
  createdAt: Date;               // When the expense was recorded
  updatedAt: Date;               // Last modification timestamp
}
```

**Constraints**:
- `amount` must be greater than 0
- `date` cannot be in the future (validation rule)
- `description` must be between 1-255 characters
- `categoryId` must reference an existing Category

### Category

Represents a classification for organizing expenses.

```typescript
interface Category {
  id: string;           // UUID - Primary identifier
  name: string;         // Human-readable category name
  color: string;        // Hex color code for UI representation
  budgetLimit?: number; // Optional monthly budget limit
  createdAt: Date;      // When the category was created
  updatedAt: Date;      // Last modification timestamp
}
```

**Constraints**:
- `name` must be unique and between 1-50 characters
- `color` must be a valid hex color code
- `budgetLimit` must be greater than 0 if provided

### PaymentMethod

Enumeration of supported payment methods.

```typescript
enum PaymentMethod {
  CASH = 'cash',
  CREDIT_CARD = 'credit_card',
  DEBIT_CARD = 'debit_card',
  BANK_TRANSFER = 'bank_transfer',
  DIGITAL_WALLET = 'digital_wallet',
  OTHER = 'other'
}
```

### Filter

Represents filtering criteria for expense queries.

```typescript
interface Filter {
  dateRange?: {
    start: Date;
    end: Date;
  };
  categoryIds?: string[];     // Array of category IDs to include
  amountRange?: {
    min: number;
    max: number;
  };
  paymentMethods?: PaymentMethod[];
  searchText?: string;        // Text search in description and notes
}
```

## Entity Relationships

```
Category (1) ──────── (N) Expense
  ▲                        ▲
  │                        │
  │              PaymentMethod (enum)
  │                        │
  └─────── Filter ─────────┘
```

- Each `Expense` belongs to exactly one `Category`
- Each `Category` can have zero or more `Expenses`
- `Filter` is a query construct that references `Category` and `PaymentMethod`

## Data Storage Schema

### IndexedDB Structure

```javascript
// Database: ExpenseTrackerDB (v1)
const stores = {
  expenses: {
    keyPath: 'id',
    indexes: [
      { name: 'date', keyPath: 'date' },
      { name: 'categoryId', keyPath: 'categoryId' },
      { name: 'amount', keyPath: 'amount' },
      { name: 'paymentMethod', keyPath: 'paymentMethod' },
      { name: 'description', keyPath: 'description' },
      { name: 'date-category', keyPath: ['date', 'categoryId'] },
      { name: 'date-amount', keyPath: ['date', 'amount'] }
    ]
  },
  categories: {
    keyPath: 'id',
    indexes: [
      { name: 'name', keyPath: 'name', unique: true }
    ]
  }
};
```

## Data Validation Rules

### Expense Validation

```typescript
const expenseValidation = {
  amount: {
    required: true,
    type: 'number',
    min: 0.01,
    max: 999999999.99
  },
  date: {
    required: true,
    type: 'date',
    max: new Date() // Cannot be in the future
  },
  categoryId: {
    required: true,
    type: 'string',
    existsIn: 'categories'
  },
  paymentMethod: {
    required: true,
    type: 'enum',
    values: Object.values(PaymentMethod)
  },
  description: {
    required: true,
    type: 'string',
    minLength: 1,
    maxLength: 255
  },
  notes: {
    required: false,
    type: 'string',
    maxLength: 1000
  }
};
```

### Category Validation

```typescript
const categoryValidation = {
  name: {
    required: true,
    type: 'string',
    minLength: 1,
    maxLength: 50,
    unique: true
  },
  color: {
    required: true,
    type: 'string',
    pattern: /^#[0-9A-F]{6}$/i // Hex color
  },
  budgetLimit: {
    required: false,
    type: 'number',
    min: 0.01
  }
};
```

## Data Access Patterns

### Common Queries

1. **Get expenses by date range**
   ```sql
   SELECT * FROM expenses 
   WHERE date >= ? AND date <= ? 
   ORDER BY date DESC
   ```

2. **Get expenses by category**
   ```sql
   SELECT * FROM expenses 
   WHERE categoryId = ? 
   ORDER BY date DESC
   ```

3. **Get category totals for date range**
   ```sql
   SELECT categoryId, SUM(amount) as total
   FROM expenses 
   WHERE date >= ? AND date <= ? 
   GROUP BY categoryId
   ```

4. **Search expenses by text**
   ```sql
   SELECT * FROM expenses 
   WHERE description LIKE ? OR notes LIKE ?
   ORDER BY date DESC
   ```

## Performance Considerations

### Indexing Strategy

- Primary indexes on `id` for all entities
- Date indexes for time-based queries
- Composite indexes for common filter combinations
- Full-text search index for description and notes

### Pagination

For large datasets, implement cursor-based pagination:

```typescript
interface PaginationParams {
  limit: number;
  cursor?: string; // Expense ID to start after
  direction: 'forward' | 'backward';
}
```

### Caching Strategy

- Cache frequently accessed categories in memory
- Implement LRU cache for filtered expense queries
- Cache chart data calculations for common date ranges

## Data Migration

### Versioning Strategy

Database versioning will handle schema migrations:

```typescript
const migrations = {
  1: {
    up: (db) => {
      // Create initial schema
    },
    down: (db) => {
      // Revert to previous version
    }
  },
  2: {
    up: (db) => {
      // Add new indexes or fields
    },
    down: (db) => {
      // Remove version 2 changes
    }
  }
};
```

## Data Export/Import

### Export Format

```json
{
  "version": "1.0",
  "exportedAt": "2025-10-24T02:20:00.000Z",
  "data": {
    "categories": [...],
    "expenses": [...]
  }
}
```

### Import Validation

- Validate data structure against current schema version
- Handle version compatibility and data transformation
- Provide detailed error reporting for invalid data