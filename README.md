# Cricket Match Ticket Booking System Design

This system facilitates advanced ticket bookings for cricket matches, ensuring a smooth user experience while managing high levels of concurrent usage.

## Requirements

### Functional Requirements

- User Registration and Authentication.
- B2C users can explore upcoming matches and filter by date, team, and venue.
- Allows B2C end users to book, view and cancel bookings.
- Third party payment integrations (Support for payment methods credit/debit cards).
- SMS and Email notifications.
- Atomic transactions to ensure that only one customer can reserve a specific seat for a match.

### Non-Functional Requirements

- The system must support between 5Lac to 1 million concurrent users.
- It should be highly scalable, available, reliable, consistent, and fault-tolerant.
- User data must be encrypted during both transmission and storage.
- Secure authentication protocols must be implemented (e.g., OAuth, JWT).
- The codebase should be modular and thoroughly documented to enable easy future updates and maintenance.
- Regular data backup procedures must be in place.
- Comprehensive monitoring and logging mechanisms should be established.
- Cost-effectiveness should be a key consideration.

## Thesis
This system is built entirely on AWS. Utilizing a microservices architecture where each component operates independently. App servers and workers queues are hosted on ec2 with media hosted on S3 and deployment managed through Docker. Logging is handled.

### Estimated Requests per Second

Assuming an average of **2 requests per user per second**, the system should be capable of handling the following:

- **For 1 million concurrent users**:
  1,000,000 users * 2 requests/user/second = 2,000,000 requests/second

## Booking Mechanism
To avoid multiple users booking the same seat, a timeout-based blocking mechanism temporarily reserves contested seats for a defined period (e.g., 10 minutes). The first user to initiate the reservation is prioritized. If payment is not confirmed within the allocated time, the seats are released for other users. This method is implemented using a Redis-based approach with a defined TTL.

## Diagram
![Diagram](https://iili.io/2HyIMxt.png)

## System Components

Each element plays a vital role in ensuring a seamless user experience, particularly during high-demand scenarios such as ticket sales for popular matches. By carefully designing the architecture with these components, the system will be well-prepared to handle the challenges of real-time ticket booking.

**CDN:** We use CDN to ensure faster delivery of media (images/videos) and reduce latency. Can handle high traffic loads during peak times, such as when tickets go on sale for a popular match. By caching content and minimizing the load on the origin server, CDNs can help reduce bandwidth costs. Modern CDNs also provide many security features against possible threats such as DDoS.

**Load Balancer:** We use a load balancer to distribute load across servers and ensure no single server becomes overwhelmed. This allows us to add or remove servers based on traffic demand, keeping the system highly concurrent and scalable. Also, High Availability: If one server goes down, the load balancer redirects traffic to other healthy servers. Session Persistence.

**App Servers:** We have a horizontally scaled distributed network of app servers running different containerized microservices within them to handle a lot of requests in parallel. It ensures smooth running of the business.

**Database:** Since the booking process is quite essentially a transactional one, we need to enforce strict ACID compliance. Additionally, we need to store many other relational information about operators, stadiums, seats, etc.To achieve both goals, we have used PostgreSQL in this exercise. We can configure partitions based on regions if the user base grows. Consistent hashing can be used for achieve data sharding. Replication can be similar to Redis. Go with master-master-slave architecture where slaves can be used for reading as read nodes and master can be used for writes. That way, we can easily scale RDBMS as well.

We have tons of data like show-related info, such as reviews, and these cannot be saved in RDBMS because this can be considered big data or too much data for RDBMS to digest. So, it can actually use any kind of NoSQL DB where we are distributed by geo, like Cassandra, which is good at holding this type of big data. It is distributed across geo, and we can set the replication factor to specify how many copies of the same data can be saved and also the consistency level. That way, you can always be sure that we won't lose data and maintain 99.99% availability and durability (so if two nodes go down, we still have the data in other nodes).

**Elasticsearch:** Used to support the search APIs on our platform (to search matches). Elasticsearch is distributed and it has RESTful search APIs available in the system. It can also be used as an analytics engine that works as an app-level search engine to answer all the search queries from the front end.

**Cache:** Utilizes Redis or Memcached to store match-related information for rapid access.

**Message Queue:** Adopts Kafka for asynchronous notifications. The notification service will generate messages and publish them to the queue, while consumers of the queue will be responsible for delivering emails, SMS, and push notifications to the intended recipients in the background without blocking user interactions.

**HADOOP (OLAP):** Manages the Business Intelligence pipeline, handling structured and unstructured data related to user activity, bookings, cancellations, and other relevant information for analytics and machine learning models.

**Continuous Integration/Continuous Deployment (CI/CD):** Automation enables rapid deployment of new features and fixes, improving system agility.

## Microservices Breakdown for Cricket Match Ticket Booking System

### User Service
Handles user registration, authentication, and profile management, ensuring secure access and user identity management.

### Search Service
Interacts with Elasticsearch via RESTful APIs to retrieve data. It supports various customizations, including filters and ongoing matches, enhancing the search experience.

### Booking Service
Serves as the core component managing the entire booking workflow. Responsibilities include fetching seat availability, reserving seats, initiating payment processes, and confirming bookings upon successful transactions.

### Ticket Service
Manages ticket inventory, encompassing availability, pricing, and types of tickets. It handles reservations and purchases, and triggers workflows for billing, notifications, and invoicing related to both bookings and cancellations.

### Payment Service
Oversees all payment-related workflows, interfacing with various payment gateways to facilitate secure and efficient financial transactions.

### Notification Service
Delivers notifications to users via email and SMS, providing updates on bookings, reminders, and changes in match status.

### Review Service
Enables users to submit reviews and ratings for matches and venues, fostering community feedback and engagement.

### Analytics Service
Collects and analyzes data on ticket sales, user behavior, and match popularity to inform business strategies and improve user experience.

### Admin Service
Equips administrators with tools for managing matches, users, and ticket sales. It also monitors system health and performance metrics for operational oversight.

### Billing Service
Maintains the organizational ledger for every transaction within the system. This service is vital for the finance team to ensure accurate and timely payouts to relevant party.

### Inventory Novel Service
Ensures the accuracy of search data by updating Elasticsearch with any modifications (creation, updates, or deletions) made to match information, keeping the inventory data current.

## Database tables (Covered main tables and fields only):

- **Users** can have multiple **Bookings**.
- **Cities** can host multiple **Teams** and **Matches**.
- **Teams** are associated with specific **Matches**.
- Each **Match** can have multiple **Tiers**, representing different pricing levels.
- **Seats** belong to specific **Tiers** and can be booked.
- **Bookings** are linked to both **Users** and **Matches**, and can include multiple **Tickets**.
- Each **Ticket** is tied to a specific **Seat** and is linked to a **Booking**.
- **Payments** are associated with **Bookings** to track financial transactions.

![DB](https://i.postimg.cc/zDgKmjG4/DB.png)