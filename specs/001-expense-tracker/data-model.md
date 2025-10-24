# Data Model: Expense Tracker Application

## Overview

This document defines the data model and entity relationships for the expense tracker application. The model is designed to support efficient storage, retrieval, and analysis of financial transactions while maintaining data integrity and performance.

## Core Entities

### Expense

Represents a single financial transaction recorded by the user.

```javascript
class Expense {
  constructor(data) {
    this.id = data.id || this.generateId();
    this.amount = data.amount;
    this.date = new Date(data.date);
    this.categoryId = data.categoryId;
    this.paymentMethod = data.paymentMethod;
    this.description = data.description;
    this.notes = data.notes || null;
    this.createdAt = data.createdAt ? new Date(data.createdAt) : new Date();
    this.updatedAt = data.updatedAt ? new Date(data.updatedAt) : new Date();
  }
  
  generateId() {
    return 'exp_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
}
```

**Constraints**:
- `amount` must be greater than 0
- `date` cannot be in the future (validation rule)
- `description` must be between 1-255 characters
- `categoryId` must reference an existing Category

### Category

Represents a classification for organizing expenses.

```javascript
class Category {
  constructor(data) {
    this.id = data.id || this.generateId();
    this.name = data.name;
    this.color = data.color;
    this.budgetLimit = data.budgetLimit || null;
    this.createdAt = data.createdAt ? new Date(data.createdAt) : new Date();
    this.updatedAt = data.updatedAt ? new Date(data.updatedAt) : new Date();
  }
  
  generateId() {
    return 'cat_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }
}
```

**Constraints**:
- `name` must be unique and between 1-50 characters
- `color` must be a valid hex color code
- `budgetLimit` must be greater than 0 if provided

### PaymentMethod

Enumeration of supported payment methods.

```javascript
const PaymentMethod = {
  CASH: 'cash',
  CREDIT_CARD: 'credit_card',
  DEBIT_CARD: 'debit_card',
  BANK_TRANSFER: 'bank_transfer',
  DIGITAL_WALLET: 'digital_wallet',
  OTHER: 'other'
};
```

### Filter

Represents filtering criteria for expense queries.

```javascript
class Filter {
  constructor(options = {}) {
    this.dateRange = options.dateRange || null;
    this.categoryIds = options.categoryIds || [];
    this.amountRange = options.amountRange || null;
    this.paymentMethods = options.paymentMethods || [];
    this.searchText = options.searchText || '';
  }
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

### SQLite Database Structure

```sql
-- Database: expense_tracker.db (v1)

-- Categories table
CREATE TABLE categories (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL UNIQUE,
  color TEXT NOT NULL,
  budgetLimit REAL,
  createdAt DATETIME NOT NULL,
  updatedAt DATETIME NOT NULL
);

-- Expenses table
CREATE TABLE expenses (
  id TEXT PRIMARY KEY,
  amount REAL NOT NULL,
  date DATETIME NOT NULL,
  categoryId TEXT NOT NULL,
  paymentMethod TEXT NOT NULL,
  description TEXT NOT NULL,
  notes TEXT,
  createdAt DATETIME NOT NULL,
  updatedAt DATETIME NOT NULL,
  FOREIGN KEY (categoryId) REFERENCES categories(id) ON DELETE RESTRICT
);

-- Indexes for performance
CREATE INDEX idx_expenses_date ON expenses(date);
CREATE INDEX idx_expenses_categoryId ON expenses(categoryId);
CREATE INDEX idx_expenses_amount ON expenses(amount);
CREATE INDEX idx_expenses_paymentMethod ON expenses(paymentMethod);
CREATE INDEX idx_expenses_date_category ON expenses(date, categoryId);
CREATE INDEX idx_expenses_date_amount ON expenses(date, amount);
CREATE INDEX idx_categories_name ON categories(name);
```

## Data Validation Rules

### Expense Validation

```javascript
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

```javascript
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
   SELECT e.categoryId, c.name as categoryName, c.color, SUM(e.amount) as total, COUNT(e.id) as count
   FROM expenses e
   JOIN categories c ON e.categoryId = c.id
   WHERE e.date >= ? AND e.date <= ?
   GROUP BY e.categoryId, c.name, c.color
   ```

4. **Search expenses by text**
   ```sql
   SELECT * FROM expenses
   WHERE description LIKE ? OR notes LIKE ?
   ORDER BY date DESC
   ```

5. **Get expenses with category details**
   ```sql
   SELECT e.*, c.name as categoryName, c.color as categoryColor
   FROM expenses e
   JOIN categories c ON e.categoryId = c.id
   ORDER BY e.date DESC
   ```

## Performance Considerations

### Indexing Strategy

- Primary indexes on `id` for all entities
- Date indexes for time-based queries
- Composite indexes for common filter combinations
- Full-text search index for description and notes

### Pagination

For large datasets, implement cursor-based pagination:

```javascript
class PaginationParams {
  constructor(options = {}) {
    this.limit = options.limit || 50;
    this.cursor = options.cursor || null;
    this.direction = options.direction || 'forward';
  }
}
```

### Caching Strategy

- Cache frequently accessed categories in memory
- Implement LRU cache for filtered expense queries
- Cache chart data calculations for common date ranges
- Use prepared statements for repeated queries

## Data Migration

### Versioning Strategy

Database versioning will handle schema migrations:

```javascript
const migrations = {
  1: {
    up: (db) => {
      // Create initial schema
      db.exec(`
        CREATE TABLE categories (...);
        CREATE TABLE expenses (...);
        CREATE INDEX ...;
      `);
    },
    down: (db) => {
      // Revert to previous version
      db.exec(`DROP TABLE IF EXISTS expenses; DROP TABLE IF EXISTS categories;`);
    }
  },
  2: {
    up: (db) => {
      // Add new indexes or fields
      db.exec(`ALTER TABLE expenses ADD COLUMN tags TEXT;`);
    },
    down: (db) => {
      // Remove version 2 changes
      db.exec(`ALTER TABLE expenses DROP COLUMN tags;`);
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