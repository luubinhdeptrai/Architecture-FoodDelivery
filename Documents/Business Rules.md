**Business Rules for Food Delivery Platform**

**1. Account & Access Rules**

* **BR-1 (Partner Verification):** System administrators must manually verify and approve newly registered Restaurant Partners and Delivery Personnel (Shippers) before they are permitted to operate or receive orders on the platform.

**2. Ordering & Shopping Cart Rules**

* **BR-2 (Single-Restaurant Cart Constraint):** A customer is strictly prohibited from adding menu items from multiple different restaurants into a single shopping cart simultaneously. Customers must ensure all items in their cart are from a single restaurant before proceeding to checkout.
* **BR-3 (Delivery Radius Constraint):** The system shall only permit a customer to successfully submit an order if their specified delivery address falls within the active restaurant's designated operational radius.

**3. Payment & Financial Rules**

* **BR-4 (Cash on Delivery Exclusivity):** For the purposes of Release 1 (MVP), Cash on Delivery (COD) is the sole supported and permitted payment method for all transactions. Integration with online payment gateways (VNPay, MoMo) is explicitly deferred to subsequent releases.
* **BR-5 (Commission Calculation):** The platform calculates its expected commission from restaurants as a fixed percentage of the Gross Merchandise Value (GMV), which is determined by the total COD collected by shippers for that specific restaurant.

**4. Operational & Fulfillment Rules**

* **BR-6 (Geographical Scope Constraint):** For the initial MVP, the software's operational usage and order fulfillment are geographically restricted to a single, pre-designated service area within Vietnam.
* **BR-7 (Order Lifecycle Integrity):** All orders must transition sequentially through specific states to ensure tracking accuracy: Pending -> Accepted -> Preparing -> Ready for Pickup/Picked Up -> Delivered.
* **BR-8 (Real-time Availability Control):** Restaurant staff possess the authority to mark specific menu items or their entire restaurant as "Unavailable/Sold Out/Closed". When triggered, this rule immediately updates the customer-facing database, blocking any new users from adding the affected items to their carts.
* **BR-9 (Enterprise Exclusion):** The platform does not support B2B enterprise ordering or subscription-based meal plans.