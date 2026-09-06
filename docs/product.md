# Product Requirements Document: Shopping Cart (MVP)

## 1. Product objective

Provide a lightweight backend service that allows customers to discover products and manage a shopping cart. The MVP focuses on the core browse-and-cart experience and intentionally excludes checkout, payment, and order fulfillment.

## 2. Actors

- **Customer (Shopper)**: browses products, creates a cart, adds/updates/removes/clears items, and views the running total.

## 3. MVP capabilities

- Browse the list of products.
- View details of a single product.
- Create or retrieve an active shopping cart.
- Add a product to the cart.
- Update the quantity of an item in the cart.
- Remove an item from the cart.
- Clear the cart.
- View the cart and its total.

## 4. Functional requirements

### Browse products
- The system shall expose a list of products.
- Each product summary shall include at minimum: id, name, and price.

### View a product
- The system shall return product details for a given product id, including a primary image.
- If the product does not exist or is inactive, the system shall return a not-found indication.

### Create a cart
- The system shall return an existing active cart for the customer if one exists; otherwise, it shall create a new, empty cart and return a cart identifier.

### Add to cart
- Given a cart id and a product id, the system shall add the product to the cart.
- The system shall reject attempts to add an inactive product to the cart.
- If the product is already in the cart, the system shall increase the quantity by the requested amount, up to a maximum line-item quantity of 99.
- The system shall reject quantities that are less than 1 or would cause the line-item quantity to exceed 99.

### Update item quantity
- Given a cart id, product id, and a new quantity, the system shall set the item quantity to the requested value.
- If the quantity is zero, the item shall be removed from the cart.
- The system shall reject quantities that are negative or greater than 99.
- The system shall reject any update that increases the quantity of an inactive product.

### Remove item
- Given a cart id and a product id, the system shall remove the item from the cart entirely.

### Clear cart
- Given a cart id, the system shall remove all line items from the cart.
- The cart identifier remains valid and the cart remains available for further use.

### View cart
- The system shall return the list of items in the cart, including products that have become inactive, each with product summary, current unit price, quantity, and line total.
- The system shall return the cart total, computed as the sum of line totals.

## 5. Business rules

- A cart contains zero or more line items.
- A customer may have only one active cart at a time.
- Each line item references one product and a quantity between 1 and 99, inclusive.
- Only active products can be returned by the product list and newly added to a cart.
- Products that become inactive after being added to a cart remain visible; their quantity may be reduced or the item removed, but increasing the quantity is not allowed.
- Adding an existing product to a cart increments its quantity, up to a maximum of 99.
- Updating an item quantity to zero removes the line item from the cart.
- Line totals and the cart total are calculated using each product's current price.
- There is no maximum number of distinct products in a cart.
- Clearing a cart removes all line items; the cart itself is not deleted.

## 6. Assumptions

- The product catalog is pre-populated and read-only for the MVP.
- Each product has one primary image. Multiple images are future scope.
- A single currency is used; no currency conversion is required.
- Product availability is represented only by an active/inactive flag.
- Customers are anonymous. Authentication is not required.
- Carts do not expire automatically in the MVP.
- Concurrent updates to the same cart are out of scope; last-write-wins behavior is acceptable.

## 7. Out-of-scope items

- Authentication and authorization
- Payments and payment gateways
- Order creation, order history, and order status tracking
- Shipping calculations, shipping methods, and delivery tracking
- Inventory and stock management (stock quantities, reservation, decrement, and related behavior)
- Multiple product images
- Product catalog management or administration
- Coupons, discounts, gift cards, and promotional pricing
- Notifications (email, SMS, push)
- Microservices architecture
- Kafka, Redis, or external message brokers/caches
- Cart expiration, idle timeout, and abandoned-cart cleanup
- Product-list pagination, filtering, sorting, and search

## 8. Open questions

- Where will product catalog data come from and how will it be seeded for development and testing?
- What is the technical mechanism for identifying an anonymous customer and their active cart?
- What is the intended persistence strategy for carts (in-memory, relational database, file-based)?
- What rounding rules apply to monetary totals?
