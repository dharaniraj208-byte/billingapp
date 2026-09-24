# Shop Billing Manager

A simple shop management web app for secure billing, product and inventory tracking, first-time shop setup, sales dashboards, invoices, and editable business settings.

## Generated with GenMB

This project was generated using [GenMB](https://genmb.com) - AI-powered application builder.

### Original Prompt

> Absolutely. You can use the following as a **detailed prompt for an AI website builder** such as Lovable, Bolt, Replit, or another coding AI:

Build a modern, simple, user-friendly **Shop Billing and Product Management Website** for small and medium-sized shops.

## 1. Login and First-Time Setup

Create a secure login page for the shop owner/user.

When a user logs in for the first time, show a **Shop Setup page** where they can enter:

* Shop Name
* Owner Name
* Mobile Number
* Email Address
* Shop Address
* GST Number (optional)
* Shop Logo (optional)

Save these details in the database.

The **Shop Name should automatically appear at the top of every bill/invoice**.

Allow the user to edit shop details later from a **Settings** page.

## 2. Main Dashboard

After login, show a clean dashboard containing:

* Today's Total Sales
* Number of Bills Today
* Total Products
* Low Stock Products
* Recent Bills
* Quick "Create New Bill" button
* Quick "Add Product" button

Use a clean and simple interface that can be easily understood by a shop owner who is not very technical.

## 3. Product Management

Create a **Products** section where the user can add and manage products.

Add Product form should contain:

* Product Name
* Product Code / SKU
* Category
* Purchase Price
* Selling Price
* Current Stock Quantity
* Minimum Stock Alert Quantity
* Unit (piece, kg, litre, box, etc.)
* Product Image (optional)

Buttons:

* Add Product
* Edit Product
* Delete Product
* Search Product
* View Product

The product list should have:

* Product Name
* Category
* Selling Price
* Stock Quantity
* Stock Status
* Edit button
* Delete button

Add a search box so the shop owner can quickly find products by name or product code.

## 4. Create Customer Bill

Create a very easy-to-use **Billing / POS page**.

At the top display:

**SHOP NAME**

Below the shop name display:

* Shop Address
* Mobile Number
* GST Number if available

Then provide a customer section:

* Customer Name (optional)
* Customer Mobile Number (optional)

### Add Products to Bill

The user should be able to quickly add products to the bill.

Provide a large **Search Product** box.

When the user types the product name or product code, show matching products.

When the user selects a product, automatically add it to the bill.

Each bill item should contain:

* Product Name
* Quantity
* Selling Price
* Discount
* Total Price
* Remove button

Make quantity editing extremely easy.

For example:

Product: Rice 5kg
Quantity: [ 2 ]
Price: ₹500
Total: ₹1,000

Allow the user to increase/decrease quantity using **+ and - buttons**.

Also allow the user to manually type the quantity.

Automatically calculate:

* Subtotal
* Discount
* Tax/GST if enabled
* Grand Total

## 5. Stock Management

When a bill is completed, automatically reduce the sold quantity from the product's stock.

Example:

Available stock = 20

Customer buys = 3

New stock = 17

Do not allow the user to sell more quantity than the available stock unless an **Allow Negative Stock** option is enabled in Settings.

Show a warning when stock reaches the minimum stock level.

Example:

**Low Stock: Rice 5kg – Only 4 remaining**

## 6. Payment Section

At the bottom of the billing page provide:

* Cash
* UPI
* Card
* Credit / Due

Allow the user to select the payment method.

Show:

**Total Amount: ₹1,250**

Then provide:

**Amount Received: ₹1,500**

Automatically calculate:

**Change: ₹250**

For credit/due payments, store the customer's outstanding amount.

## 7. Generate Invoice

After clicking **Complete Bill**, generate a professional invoice.

Invoice should contain:

* Shop Logo
* Shop Name
* Shop Address
* Phone Number
* GST Number
* Invoice Number
* Date and Time
* Customer Name
* Customer Phone
* Product Name
* Quantity
* Price
* Discount
* Tax
* Total
* Payment Method
* Grand Total

At the bottom show:

**Thank you! Visit Again**

Provide buttons:

* Print Invoice
* Download PDF
* New Bill

Make the invoice suitable for both **A4 printing and small thermal receipt printers**.

## 8. Billing History

Create a **Bills / Sales History** page.

Show:

* Invoice Number
* Date
* Customer Name
* Total Amount
* Payment Method
* View Bill
* Print
* Download PDF

Add filters for:

* Today
* Yesterday
* This Week
* This Month
* Custom Date Range

Add a search option for invoice number or customer name.

## 9. Customer Management

Create a **Customers** section.

Store:

* Customer Name
* Mobile Number
* Address
* Total Purchases
* Outstanding/Due Amount

When creating a bill, the user can select an existing customer or create a new customer.

Show the customer's previous bills and payment history.

## 10. Reports

Create a Reports section containing:

* Daily Sales Report
* Weekly Sales Report
* Monthly Sales Report
* Product Sales Report
* Profit Report
* GST/Tax Report
* Payment Method Report
* Outstanding/Due Report

Display important information using simple charts and tables.

## 11. Settings

Create a Settings page with:

### Shop Settings

* Shop Name
* Owner Name
* Address
* Phone
* Email
* GST Number
* Logo

### Billing Settings

* Invoice Prefix
* Starting Invoice Number
* Default Tax
* Default Discount
* Currency
* Enable/Disable GST

### Inventory Settings

* Low Stock Alert
* Allow Negative Stock

### User Settings

Allow the shop owner to change their login details and password.

## 12. UI/UX Requirements

The website must be:

* Very simple
* Fast
* Mobile responsive
* Desktop responsive
* Tablet responsive
* Easy for shop owners to use
* Clean and professional

Use large buttons and readable fonts.

The **Create Bill** page should be the easiest and fastest part of the application.

Use keyboard-friendly controls so the shop owner can quickly search products, enter quantity, and complete a bill.

Use confirmation dialogs before deleting products or bills.

Show clear success/error messages such as:

**Product added successfully**

**Bill generated successfully**

**Insufficient stock**

## 13. Navigation

Create a sidebar/menu containing:

1. Dashboard
2. Create Bill
3. Products
4. Customers
5. Bills
6. Reports
7. Settings
8. Logout

On mobile, convert the sidebar into a mobile-friendly menu.

## 14. Database

Create a proper database with relationships between:

* Users
* Shops
* Products
* Customers
* Bills
* Bill Items
* Payments
* Stock Transactions

Each shop/user should only be able to see their own products, customers, bills, and reports.

## 15. Important Billing Logic

Implement the following automatically:

* Product price should be retrieved when a product is selected.
* Quantity changes should immediately update the line total.
* Subtotal should update automatically.
* Discount should update automatically.
* Tax should update automatically.
* Grand total should update automatically.
* Stock should decrease after successful bill completion.
* Invoice number should automatically increase.
* Bill date and time should be automatically recorded.
* Payment method should be stored.
* Customer due amount should be updated when credit is used.

## 16. Overall Goal

The final website should feel like a **simple POS billing application for a real local shop**, not like a complicated accounting system.

The main workflow should be:

**Login → Dashboard → Create Bill → Search Product → Select Product → Enter Quantity → Add More Products → Select Payment → Complete Bill → Print/Download Invoice**

Prioritize **speed, simplicity, accuracy, and ease of use**.

Build the complete frontend, backend, database, authentication, billing calculations, inventory management, invoice generation, and responsive UI.


## Getting Started

### Prerequisites

- Node.js 18+

### Running Locally

```bash
npm install
npm run dev
```

## Framework

This project uses **React-Ts**.

## Progressive Web App (PWA)

This app is PWA-enabled and can be installed on mobile devices!

### PWA Files Included

- `manifest.json` - App manifest for installability
- `service-worker.js` - Caching and offline support
- `offline.html` - Offline fallback page
- `install-prompt.js` - "Add to Home Screen" install banner

### Installing on Mobile

1. Open the deployed app in your mobile browser
2. A custom install banner will appear after 2 seconds
3. Tap "Install" to add the app to your home screen
4. On iOS: Tap the share button and select "Add to Home Screen" (iOS shows instructions)

### Testing PWA Locally

PWA features require HTTPS to work. For local testing:

```bash
# Option 1: Use a local HTTPS server
npx local-web-server --https

# Option 2: Use Chrome's DevTools
# Open DevTools > Application > Service Workers
# Check "Bypass for network" to test offline mode
```

## License

MIT
