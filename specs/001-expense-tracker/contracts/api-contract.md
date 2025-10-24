# API Contracts: Expense Tracker Application

## Overview

This document defines the contracts and interfaces for the expense tracker application's data layer. These contracts ensure consistency between components and enable proper testing and maintenance.

## Core Interfaces

### IStorageService

Defines the contract for all data storage operations.

```typescript
interface IStorageService {
  // Initialize the storage system
  initialize(): Promise<void>;
  
  // Expense operations
  getExpenses(filter?: Filter): Promise<Expense[]>;
  getExpense(id: string): Promise<Expense | null>;
  saveExpense(expense: Omit<Expense, 'id' | 'createdAt' | 'updatedAt'>): Promise<Expense>;
  updateExpense(id: string, updates: Partial<Expense>): Promise<Expense>;
  deleteExpense(id: string): Promise<void>;
  
  // Category operations
  getCategories(): Promise<Category[]>;
  getCategory(id: string): Promise<Category | null>;
  saveCategory(category: Omit<Category, 'id' | 'createdAt' | 'updatedAt'>): Promise<Category>;
  updateCategory(id: string, updates: Partial<Category>): Promise<Category>;
  deleteCategory(id: string): Promise<void>;
  
  // Analytics operations
  getCategoryTotals(filter?: Filter): Promise<CategoryTotal[]>;
  getExpenseTrends(filter: Filter, groupBy: 'day' | 'week' | 'month'): Promise<ExpenseTrend[]>;
  
  // Utility operations
  exportData(): Promise<ExportData>;
  importData(data: ExportData): Promise<ImportResult>;
}
```

### IExpenseService

Defines the contract for expense business logic.

```typescript
interface IExpenseService {
  // CRUD operations
  createExpense(data: CreateExpenseData): Promise<Expense>;
  updateExpense(id: string, data: UpdateExpenseData): Promise<Expense>;
  deleteExpense(id: string): Promise<void>;
  getExpense(id: string): Promise<Expense | null>;
  getExpenses(filter?: Filter): Promise<ExpenseListResult>;
  
  // Validation
  validateExpense(data: CreateExpenseData | UpdateExpenseData): ValidationResult;
  
  // Analytics
  getExpenseSummary(filter?: Filter): Promise<ExpenseSummary>;
  getExpensesByCategory(filter?: Filter): Promise<CategoryExpense[]>;
  getMonthlyTrends(year: number): Promise<MonthlyTrend[]>;
}
```

### ICategoryService

Defines the contract for category management.

```typescript
interface ICategoryService {
  // CRUD operations
  createCategory(data: CreateCategoryData): Promise<Category>;
  updateCategory(id: string, data: UpdateCategoryData): Promise<Category>;
  deleteCategory(id: string): Promise<void>;
  getCategory(id: string): Promise<Category | null>;
  getCategories(): Promise<Category[]>;
  
  // Validation
  validateCategory(data: CreateCategoryData | UpdateCategoryData): ValidationResult;
  
  // Utility
  getCategoryUsage(id: string): Promise<CategoryUsage>;
  mergeCategories(sourceId: string, targetId: string): Promise<void>;
}
```

## Data Transfer Objects (DTOs)

### Expense DTOs

```typescript
interface CreateExpenseData {
  amount: number;
  date: Date;
  categoryId: string;
  paymentMethod: PaymentMethod;
  description: string;
  notes?: string;
}

interface UpdateExpenseData {
  amount?: number;
  date?: Date;
  categoryId?: string;
  paymentMethod?: PaymentMethod;
  description?: string;
  notes?: string;
}

interface ExpenseListResult {
  expenses: Expense[];
  totalCount: number;
  hasMore: boolean;
  nextCursor?: string;
}
```

### Category DTOs

```typescript
interface CreateCategoryData {
  name: string;
  color: string;
  budgetLimit?: number;
}

interface UpdateCategoryData {
  name?: string;
  color?: string;
  budgetLimit?: number;
}

interface CategoryUsage {
  categoryId: string;
  expenseCount: number;
  totalAmount: number;
  lastUsed: Date;
}
```

### Analytics DTOs

```typescript
interface CategoryTotal {
  categoryId: string;
  categoryName: string;
  categoryColor: string;
  amount: number;
  percentage: number;
  expenseCount: number;
}

interface ExpenseTrend {
  period: string; // ISO date string
  amount: number;
  expenseCount: number;
}

interface ExpenseSummary {
  totalAmount: number;
  expenseCount: number;
  averageExpense: number;
  categoryBreakdown: CategoryTotal[];
  paymentMethodBreakdown: PaymentMethodTotal[];
  dateRange: {
    start: Date;
    end: Date;
  };
}

interface CategoryExpense {
  categoryId: string;
  categoryName: string;
  categoryColor: string;
  expenses: Expense[];
  totalAmount: number;
  percentage: number;
}

interface MonthlyTrend {
  month: string; // YYYY-MM format
  amount: number;
  expenseCount: number;
  categoryBreakdown: CategoryTotal[];
}

interface PaymentMethodTotal {
  paymentMethod: PaymentMethod;
  amount: number;
  percentage: number;
  expenseCount: number;
}
```

### Filter DTOs

```typescript
interface FilterOptions {
  dateRange?: {
    start: Date;
    end: Date;
  };
  categoryIds?: string[];
  amountRange?: {
    min: number;
    max: number;
  };
  paymentMethods?: PaymentMethod[];
  searchText?: string;
  sortBy?: 'date' | 'amount' | 'description';
  sortOrder?: 'asc' | 'desc';
  limit?: number;
  cursor?: string;
}
```

### Import/Export DTOs

```typescript
interface ExportData {
  version: string;
  exportedAt: string;
  data: {
    categories: Category[];
    expenses: Expense[];
  };
}

interface ImportResult {
  success: boolean;
  imported: {
    categories: number;
    expenses: number;
  };
  errors: ImportError[];
  warnings: ImportWarning[];
}

interface ImportError {
  type: 'validation' | 'duplicate' | 'reference';
  message: string;
  data?: any;
}

interface ImportWarning {
  type: 'data_loss' | 'conversion';
  message: string;
  data?: any;
}
```

## Validation Contracts

```typescript
interface ValidationResult {
  isValid: boolean;
  errors: ValidationError[];
}

interface ValidationError {
  field: string;
  message: string;
  code: string;
  value?: any;
}

type ValidationCode = 
  | 'REQUIRED_FIELD'
  | 'INVALID_FORMAT'
  | 'MIN_VALUE'
  | 'MAX_VALUE'
  | 'MIN_LENGTH'
  | 'MAX_LENGTH'
  | 'UNIQUE_VIOLATION'
  | 'INVALID_DATE'
  | 'REFERENCE_NOT_FOUND'
  | 'FUTURE_DATE_NOT_ALLOWED';
```

## Event Contracts

```typescript
interface ExpenseEvent {
  type: 'expense_created' | 'expense_updated' | 'expense_deleted';
  payload: {
    expense: Expense;
    timestamp: Date;
  };
}

interface CategoryEvent {
  type: 'category_created' | 'category_updated' | 'category_deleted';
  payload: {
    category: Category;
    timestamp: Date;
  };
}

interface DataEvent {
  type: 'data_imported' | 'data_exported' | 'data_cleared';
  payload: {
    timestamp: Date;
    details?: any;
  };
}

type AppEvent = ExpenseEvent | CategoryEvent | DataEvent;

interface IEventBus {
  subscribe<T extends AppEvent>(eventType: string, handler: (event: T) => void): void;
  unsubscribe(eventType: string, handler: (event: AppEvent) => void): void;
  publish(event: AppEvent): void;
}
```

## Error Contracts

```typescript
interface AppError {
  code: string;
  message: string;
  details?: any;
  timestamp: Date;
}

type ErrorCode =
  | 'STORAGE_INITIALIZATION_FAILED'
  | 'EXPENSE_NOT_FOUND'
  | 'CATEGORY_NOT_FOUND'
  | 'VALIDATION_FAILED'
  | 'STORAGE_QUOTA_EXCEEDED'
  | 'IMPORT_FAILED'
  | 'EXPORT_FAILED'
  | 'UNKNOWN_ERROR';

interface IErrorReporter {
  report(error: AppError): void;
  getErrors(): AppError[];
  clearErrors(): void;
}
```

## Performance Contracts

```typescript
interface PerformanceMetrics {
  operation: string;
  duration: number;
  timestamp: Date;
  metadata?: any;
}

interface IPerformanceMonitor {
  startOperation(operation: string): string; // Returns operation ID
  endOperation(operationId: string, metadata?: any): void;
  getMetrics(): PerformanceMetrics[];
  clearMetrics(): void;
}
```

## Implementation Notes

### Error Handling

All service methods should handle errors consistently:
- Validate input before processing
- Catch and transform storage errors
- Return standardized error objects
- Log errors for debugging

### Performance Requirements

- All read operations should complete within 100ms for datasets up to 10,000 expenses
- Write operations should complete within 50ms
- Analytics queries should complete within 200ms
- Memory usage should not exceed 50MB for typical usage patterns

### Testing Requirements

- All interfaces should have comprehensive unit tests
- Integration tests should verify contract compliance
- Performance tests should validate timing requirements
- Error scenarios should be thoroughly tested