# Merx: Durable Fintech Orchestrator 💸

![Go](https://img.shields.io/badge/Go-1.21+-00ADD8?style=flat&logo=go)
![Temporal](https://img.shields.io/badge/Orchestration-Temporal.io-blue)
![Architecture](https://img.shields.io/badge/Pattern-Saga-orange)
![Observability](https://img.shields.io/badge/Tracing-OpenTelemetry-purple)

**Merx** is a proof-of-concept distributed transaction engine designed to demonstrate **Durable Execution** in financial systems. 

Unlike choreography-based systems (event soup), Merx uses **Centralized Orchestration** (via Temporal.io) to coordinate a complex checkout flow across three distinct microservices, ensuring data consistency even in the face of network failures or service outages.

## 🏗️ Architecture & Design

The system implements the **Saga Pattern** to manage distributed transactions without a two-phase commit (2PC).

### The Workflow (Happy Path)
1.  **Order Initiated:** Client requests a purchase.
2.  **Inventory Service:** Reserves stock (Local Transaction).
3.  **Payment Service:** Charges the credit card (Local Transaction).
4.  **Shipping Service:** Generates shipping label (Local Transaction).
5.  **Completion:** Order confirmed to client.



### Failure Recovery (Compensation)
If the **Shipping Service** is down or fails:
1.  The Orchestrator catches the error.
2.  Triggers **Compensation Workflow**:
    * **Refund Payment:** Calls Payment Service to refund the charge.
    * **Release Stock:** Calls Inventory Service to release reserved items.
3.  System returns to a consistent state (Order Failed).

## 🚀 Key Technical Features

### 1. Orchestration over Choreography
Using **Temporal (Go SDK)**, the business logic is defined as code, not hidden in message queues. This provides:
* **Visibility:** The state of every transaction is queryable in real-time.
* **Retries:** Automatic exponential backoff for transient failures.

### 2. Idempotency & Deduplication
To prevent double-spending, the **Payment Service** implements strict idempotency using `Idempotency-Key` headers. 
* *Scenario:* If the orchestrator retries a payment request due to a network timeout, the Payment Service detects the duplicate key and returns the original successful response without charging the card again.

### 3. Observability
Integrated with **OpenTelemetry** and **Jaeger** to visualize the request lifecycle. Traces propagate from the Orchestrator -> Inventory -> Payment -> Shipping.

## 🛠️ Tech Stack

* **Language:** Go (Golang)
* **Orchestrator:** Temporal.io
* **Communication:** gRPC / REST
* **Tracing:** OpenTelemetry, Jaeger
* **Storage:** PostgreSQL (Microservices DB), Cassandra/Postgres (Temporal Store)

## 📂 Project Structure

```text
/cmd
  /orchestrator    # The Temporal Worker (Workflow Definitions)
  /inventory-svc   # Microservice for Stock
  /payment-svc     # Microservice for Stripe/Mock payments
  /shipping-svc    # Microservice for Logistics
/pkg
  /interceptors    # OpenTelemetry & Idempotency Middleware
  /api             # Protobuf definitions
