# Quick Start Guide: Expense Tracker Application

## Overview

This guide provides step-by-step instructions for setting up, running, and using the expense tracker application. The application is a Progressive Web App (PWA) that works entirely in the browser using local storage for data persistence.

## Prerequisites

- Modern web browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Node.js 18+ (for development)
- npm or yarn package manager

## Installation

### For Users (Browser-based)

1. Open your web browser
2. Navigate to the application URL (provided when deployed)
3. Click "Install" or "Add to Home Screen" if available for PWA functionality

### For Developers

```bash
# Clone the repository
git clone <repository-url>
cd expense-tracker

# Install dependencies
npm install

# Start development server
npm run dev

# Open browser to http://localhost:3000
```

## Initial Setup

### First Launch

1. The application will initialize automatically on first launch
2. Default categories will be created automatically:
   - Food & Dining
   - Transportation
   - Shopping
   - Entertainment
   - Bills & Utilities
   - Healthcare
   - Other

### Creating Custom Categories

1. Click on "Categories" in the navigation menu
2. Click "Add Category" button
3. Enter category name and select a color
4. Optionally set a monthly budget limit
5. Click "Save" to create the category

## Basic Usage

### Recording an Expense

1. Click the "Add Expense" button (prominent + icon)
2. Fill in the required fields:
   - **Amount**: Enter the expense amount
   - **Date**: Select the transaction date (defaults to today)
   - **Category**: Choose from your categories
   - **Payment Method**: Select how you paid
   - **Description**: Brief description of the expense
3. Optionally add notes for additional context
4. Click "Save" to record the expense

### Viewing Expenses

1. Click "Expenses" in the navigation menu
2. Your expenses will be displayed in reverse chronological order
3. Use the search bar to find specific expenses
4. Click on any expense to view or edit details

### Filtering Expenses

1. In the Expenses view, click "Filter" button
2. Set your filter criteria:
   - **Date Range**: Select start and end dates
   - **Categories**: Choose one or more categories
   - **Amount Range**: Set minimum and/or maximum amounts
   - **Payment Methods**: Select specific payment methods
3. Click "Apply Filter" to see filtered results
4. Click "Clear Filter" to reset all filters

### Viewing Analytics

1. Click "Analytics" in the navigation menu
2. Select the time period you want to analyze
3. View different chart types:
   - **Spending by Category**: Pie chart showing category breakdown
   - **Spending Trends**: Line chart showing spending over time
   - **Payment Methods**: Bar chart showing payment method usage
4. Hover over chart elements for detailed information

## Advanced Features

### Editing Expenses

1. Click on any expense in the list to open details
2. Click the "Edit" button
3. Modify the desired fields
4. Click "Save" to update the expense

### Deleting Expenses

1. Click on an expense to open details
2. Click the "Delete" button
3. Confirm the deletion in the dialog
4. The expense will be permanently removed

### Managing Categories

1. Go to "Categories" in the navigation menu
2. Click on any category to:
   - Edit name, color, or budget limit
   - View usage statistics
   - Delete (only if no expenses are assigned)
3. To delete a category with expenses:
   - Reassign expenses to another category first
   - Then delete the empty category

### Data Export

1. Go to "Settings" in the navigation menu
2. Click "Export Data"
3. Choose export format (JSON)
4. Save the exported file to your device

### Data Import

1. Go to "Settings" in the navigation menu
2. Click "Import Data"
3. Select your previously exported file
4. Review import preview
5. Confirm import to merge data

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| N | New Expense |
| / | Search Expenses |
| F | Open Filter |
| C | Categories |
| A | Analytics |
| S | Settings |
| ESC | Close modal/dialog |

## Mobile Usage

### Touch Gestures

- **Swipe left** on expense item to delete
- **Swipe right** on expense item to edit
- **Pull down** on expense list to refresh
- **Pinch to zoom** on charts for detailed view

### PWA Features

When installed as a PWA:
- Works offline with full functionality
- Receives notifications for budget alerts
- Faster loading and smoother performance
- Accessible from device home screen

## Troubleshooting

### Common Issues

**Data not saving**
- Check browser storage permissions
- Ensure sufficient storage space available
- Try refreshing the page

**Slow performance**
- Clear browser cache
- Check for too many expenses (consider archiving old data)
- Close other browser tabs

**Charts not displaying**
- Ensure JavaScript is enabled
- Check browser compatibility
- Try a different browser

### Data Recovery

If data is lost:
1. Check browser's local storage for existing data
2. Import from your last export file
3. Contact support if no backup available

### Browser Compatibility

The application supports:
- Chrome 90+ (recommended)
- Firefox 88+
- Safari 14+
- Edge 90+

For best experience, use the latest version of your preferred browser.

## Privacy and Security

- All data is stored locally in your browser
- No data is sent to external servers
- Consider regular exports for backup
- Clear browser data to remove all expense information

## Getting Help

For additional support:
1. Check the in-app help section
2. Review the user guide
3. Visit the community forum
4. Contact support through the app

## Next Steps

After getting familiar with basic usage:

1. Set up custom categories matching your spending patterns
2. Establish monthly budgets for key categories
3. Regularly review analytics for spending insights
4. Set up routine data exports for backup
5. Explore advanced filtering for detailed analysis