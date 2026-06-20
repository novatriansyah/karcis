# Product Requirements Document (PRD): Karcis

**Product Name:** Karcis (Event Ticketing Platform)  
**Status:** Draft  
**Target Launch:** Phase 1 (MVP)  

---

## 1. Product Vision & Value Proposition
Karcis is a self-service event ticketing platform designed for speed, security, and scalability. It solves the critical problem of ticket overselling and system crashes during high-demand "ticket wars" (flash sales). 

The platform offers a reliable transaction flow for customers and detailed dashboard metrics and secure entry gate verification for event organizers (promoters).

---

## 2. Target Personas

### 2.1. Customer (Ticket Buyer)
- **Pain Points:** Long queues, site crashes, paying for tickets only to find out they were already sold out, and complex checkouts.
- **Goals:** Find events, quickly secure ticket inventory, complete payment with local payment methods, and access tickets securely on mobile.

### 2.2. Event Organizer (Promoter)
- **Pain Points:** Lack of real-time sales visibility, gate congestion due to slow ticket verification, manual ticket reconciliation, and difficulty setting up complex ticket structures.
- **Goals:** Self-create and configure events, set ticket tiers (VIP, Presale, GA), monitor live revenue, and efficiently check in thousands of attendees.

### 2.3. Gate Staff (Scanner)
- **Pain Points:** Poor internet connectivity at venues, slow scanning apps, and counterfeit tickets.
- **Goals:** Instantly check in attendees with a 1-second scan-to-feedback cycle, using standard mobile devices.

---

## 3. Product Scope & Functional Requirements

### 3.1. Customer Portal (Public Web App)
The customer facing portal must be fully responsive, SEO-optimized, and performant.

#### A. Event Browsing & UX Delight
- **Search & Filter:** Search by event name, filter by date range, category (Concert, Conference, Sports, etc.), and location/city.
- **Event Detail Page:** Displays event title, description, rich media, venue details, calendar date/time, and ticket tiers.
  - **Dynamic Stock Alerts:** Display indicators like "Selling Fast" or "Only 5 tickets left" for low-stock tiers.
  - **Calendar & Maps Integrations:** "Add to Google Calendar / Apple Calendar" buttons and a deep link to Google Maps/Waze for venue coordinates.

#### B. Booking & Checkout System (High Priority)
- **Ticket Selection:** User selects quantities for multiple ticket tiers (max 4 tickets per transaction to prevent scalping).
- **Inventory Locking:** When the user clicks "Book Now", the selected quantity is reserved/locked in Redis for **10 minutes** with an active visual countdown timer.
- **Guest Distribution:** Allow the buyer to assign unique names and email/phone numbers for *each* attendee ticket bought in a group transaction.
- **Localized Payment Options:** Support Indonesian-native payment pathways (QRIS, BCA/Mandiri/BNI Virtual Accounts, and e-wallets like GoPay/OVO/Dana) via licensed gateway integrations.

#### C. Ticket Retrieval & Mobile Convenience
- **Multi-channel Delivery:**
  - **WhatsApp Notification:** Send the secure ticket QR code and PDF link directly to the purchaser's verified WhatsApp number immediately upon payment success.
  - **Email Delivery:** PDF e-ticket with event details and QR code emailed to the purchaser and distributed guests.
- **My Tickets Dashboard:** Logged-in view showing upcoming/past events with access to the ticket QR code.
- **Offline Access UX:** Instruct users on the confirmation page to download the PDF or take a screenshot of the QR code for instant offline scanning at the gate.

#### D. Authentication & Friction-Free Access
- **Frictionless Login:** Standardize OTP (One-Time Password) via WhatsApp/Email or Google Single Sign-In (OAuth).
- **Guest Checkout (No-Login Purchase):**
  - Allow buyers to purchase tickets completely without logging in or registering, **if permitted by the event promoter settings**.
  - The buyer only provides their email and phone number (WhatsApp) on the checkout form.
  - E-tickets are dispatched directly to the provided email/WhatsApp. Accessing the gate relies entirely on these offline assets.
- **Shadow/Lazy Account Linking:**
  - Behind the scenes, the backend generates a shadow customer profile associated with the guest's email and phone number.
  - If the guest later attempts to log in using OTP matching their email/phone, the system merges their session and grants them access to their entire historical guest booking ledger.

---

### 3.2. Event Organizer Dashboard (Promoter Portal)
Organizers manage their inventory, staff, and view financials.

#### A. Event & Ticket Setup
- **Creation Form:** Form to input title, category, description, terms & conditions, venue/streaming link, start date/time, end date/time, and banner image upload.
- **Access Restrictions Toggle:** Enforce a `RequireUserLogin (boolean)` setting per event.
  - When enabled (`true`), customers must authenticate (OTP/Google OAuth) before purchasing to prevent bot attacks and enforce ticket limits per account.
  - When disabled (`false`), guest checkout is permitted to maximize conversion rates for general public events (exhibitions, galleries).
- **Inventory Tiers:** Multiple ticket categories per event:
  - Ticket name (e.g., "VIP Early Bird").
  - Price (including support for free tickets).
  - Quota/Capacity.
  - Sales start & end window.
- **Manual Ticket Issuance:** Ability to issue complimentary/guest tickets or offline cash sales manually by entering name, email, and phone, bypassing the payment gateway.

#### B. Operations & Analytics
- **Live Sales Dashboard:** Tracks total tickets sold, gross revenue, daily sales charts, and checkout conversion rates.
- **Check-in Tracker:** Bar graph indicating the percentage of guests checked in vs. total ticket holders.
- **Attendee Data Export:** One-click CSV/Excel download of customer registers containing purchase info, checked-in status, NIK, and contact details.
- **Scanner Access Keys:** Generate unique, revokable tokens for check-in staff to log in to the scanner interface without giving them full dashboard access.

---

### 3.3. Check-In Web Application (Scanner Portal)
A lightweight web app optimized for mobile browsers to scan and validate tickets.

- **Camera Scanner:** Integrated HTML5 camera scanner in-browser (no app install required).
- **Validation Engine:** Checks ticket authenticity, checks if it corresponds to the correct event, and verifies check-in status:
  - **Success (Green):** Ticket valid, check-in logged.
  - **Duplicate (Red/Yellow):** Ticket was already checked in (shows date & time of first check-in).
  - **Invalid (Red):** Fake QR code or wrong event.
- **Group/Family Check-in Support:**
  - If a group booking (up to 4 tickets) is scanned via the main booking code, load all associated ticket stubs and attendee names on one screen.
  - Allow gate staff to check in all attendees at once, or select specific guests (e.g., "Check-In 2 of 4" if parts of the group arrive late).
- **Manual Search:** Search attendees by name/email if their QR code is damaged.

---

### 3.4. Platform Administrator Portal (Internal Operations Hub)
An internal panel for platform operators to manage global settings, verify organizers, audit payments, and handle customer service escalations.

#### A. Organizer KYB Verification
- **Promoter Approval:** Review and approve/deny organizer profile registrations, verified bank accounts, and corporate documents before they can publish paid events.
- **Risk Profiles:** Set custom early payout percentage allowances (0% to 50%) based on the organizer's risk profile and contract signatures.

#### B. Dispute & Refund Management
- **Refund Console:** Trigger full or partial booking refunds, routing instructions to payment gateway workers.
- **Reconciliation Resolution:** Monitor the `OVERALLOCATED_PENDING_REFUND` ledger queue. Manual action triggers to either override and authorize ticket expansion (if venue capacity permits) or release the automated gateway refund link.

#### C. System Accounting Ledger
- **Platform Ledger View:** Access real-time platform metrics, including total transaction volumes, collected platform fees (PPN tax invoices), pending payouts, and escrow balances.
- **Global Settings:** Adjust platform default transaction commission rates and platform-wide blocklists.

---

## 4. Key Logical Data Model

```
+------------------+         +------------------+
|      User        |         |      Event       |
|------------------|         |------------------|
| ID (UUID)        |         | ID (UUID)        |
| Email            | 1     * | Title            |
| PasswordHash     |<--------| OrganizerID      |
| Role (Admin/Orga)|         | Description      |
+------------------+         | Venue            |
                             | StartTime        |
                             | EndTime          |
                             +------------------+
                                       | 1
                                       |
                                       | *
                             +------------------+
                             |   TicketTier     |
                             |------------------|
                             | ID (UUID)        |
                             | EventID          |
                             | Name             |
                             | Price            |
                             | MaxQuota         |
                             | AvailableQuota   |
                             +------------------+
                                       | 1
                                       |
                                       | *
+------------------+         +------------------+
|     Booking      |         |   TicketItem     |
|------------------|         |------------------|
| ID (UUID)        | 1     * | ID (UUID)        |
| CustomerID       |<--------| BookingID        |
| TotalPrice       |         | TicketTierID     |
| PlatformFee      |         | AttendeeName     |
| OrganizerRevenue |         | QRCodeToken      |
| Status (Pending/ |         | CheckedInAt      |
|    Paid/Expired) |         +------------------+
| ReservedUntil    |
+------------------+
```

---

## 5. Non-Functional Requirements & Best Practices

To ensure system reliability, integrity, and escrow safety, the application must adhere to the following best practice solutions for known high-traffic and e-commerce edge cases.

### 5.1. Concurrency Bottlenecks (The "Ticket War" Problem)
- **Problem:** 50,000 users attempt to check out simultaneously for an event with only 5,000 available seats, leading to overselling or massive database lock queues.
- **Best Practice Solution (Decouple Inventory from Order Creation):**
  - Never use SQL queries like `SELECT COUNT(*)` or database-level transactions to verify stock on live checkout hits.
  - Maintain an atomic counter inside Redis using a TTL-based lease. When a user selects a ticket, decrement the Redis counter immediately (`DECRBY`).
  - If successful, grant them a 10-minute hold token (lease).
  - The system allows the user to proceed to the database order creation phase only if they possess a valid Redis hold token.
  - If the token expires before payment completes, a background worker increments the Redis counter back to release the stock.

### 5.2. Mass Event Cancellation (The "Scam/Force Majeure" Problem)
- **Problem:** An organizer lists an event, collects Rp 500,000,000 in ticket sales, cancels the event a week before the show, and disappears with the funds.
- **Best Practice Solution (Escrow Holding Logic + Automated Disbursal Delay):**
  - Implement a strict **T+3 Post-Event Settlement** policy. Ticket sales accumulate in the organizer's managed sub-account (e.g. Xendit) but remain locked for withdrawal.
  - Funds are unlocked only 3 business days after the event successfully concludes.
  - **The Exception (Upfront Cost Payouts):** Established, verified organizers requiring upfront capital for venue deposits can request early payout of up to 30-50% of current sales. This is subject to a manual admin review tier based on their risk profile and requires a physical contract signature.

### 5.3. Asynchronous Expiry (The "Pending Bank Transfer" Problem)
- **Problem:** A customer selects Virtual Account (VA) as their payment method, generating a 10-minute expiry window. They pay at exactly minute 09:59, but the bank's processing network delays the callback webhook to our Go backend until minute 10:05—after our application has already expired the order and put the seat back on sale.
- **Best Practice Solution (Grace Period + Over-allocation Ledger Handling):**
  - Program an explicit **5-minute buffer zone** into the backend webhook processing engine.
  - If a payment webhook arrives within 5 minutes after the database status changed to `EXPIRED`, check if the payment timestamp (`completed_at` from the payment gateway) occurred before the strict expiration time.
  - If the payment was legally on time but the webhook arrived late, change the database status to `PAID` and issue the ticket.
  - If the seat was already sold to someone else during that delay, the system must not crash or throw errors. Instead, flag the order under a specialized `OVERALLOCATED_PENDING_REFUND` state, notify the admin dashboard immediately, and trigger an automated payout refund link directly to the customer explaining the inventory conflict.

### 5.4. Webhook Reliability (The "Network Drop" Problem)
- **Problem:** The payment gateway fires a successful payment webhook, but our server encounters a brief network blip, drops the request, and returns a 502 Bad Gateway. The gateway assumes our server missed it, while the customer's money is gone but no ticket is issued.
- **Best Practice Solution (Idempotency Keys + Polling Fallback Workers):**
  - The webhook receiver endpoint must be entirely idempotent. Store every incoming external transaction ID in a database table with a unique index constraint.
  - If the gateway retries the same webhook, the backend must detect the duplicate ID, reply with `200 OK` instantly, and prevent duplicate database state mutations.
  - Set up a cron task or background worker in Go that runs every 15 minutes. It queries the database for any orders stuck in a `PENDING_PAYMENT` state for over 30 minutes, executes a direct GET request to the payment gateway's payment status API to check the ground truth, and reconciles the order state automatically.

### 5.5. Automated QR Code PDF Generation & Notification Infrastructure
- **Problem:** When a transaction hits `payment.succeeded`, the main Go API should not block the HTTP thread waiting to compile PDFs and fire email or WhatsApp messages, which degrades endpoint response times and leads to request timeouts.
- **Best Practice Solution (Decoupled Worker Pattern):**
  - Offload PDF and notification processes entirely using a decoupled worker pattern via RabbitMQ, Apache Kafka, or Redis Streams.
  - The Go API publishes a clean JSON event (`ticket.paid`) to the queue. A separate Go background worker processes this event asynchronously:
    - Generates a type-safe v4 UUID or a unique, encrypted ticket token.
    - Signs this token into a standard JSON Web Token (JWT) containing basic payload metadata (e.g. `{ "ticket_id": "xyz", "event_id": "abc" }`) using an internal private key.
    - Encodes this signed token string directly into a high-density QR code.
    - Pipes the layout template to a high-speed canvas tool or localized template renderer to yield a clean PDF ticket receipt.
    - Pushes the final asset link to Mekari Qiscus or Twilio APIs to shoot a secure WhatsApp confirmation containing the admission attachment.

### 5.6. On-Site Hardware-Driven Gates & Offline Validation Mechanics
- **Problem:** If a stadium has 20,000 attendees rushing physical gates, the scanning system cannot rely on synchronous internet lookups for every single QR scan. If the venue's cellular network drops (which happens consistently during packed events), the entire check-in line stalls.
- **Best Practice Solution (Local Hybrid-Caching with Cryptographic Verification):**
  - Tickets are generated using asymmetric cryptography (signed via Ed25519 or RS256 keys), allowing the organizer's scanning Progressive Web App (PWA) to verify if a ticket is fundamentally authentic completely offline. The scanning web app holds the matching public key.
  - **Double-Scan Prevention:** To prevent duplicates or anti-counterfeiting offline, the scanning devices pool locally via an on-site local area network (LAN) router using a localized Redis instance hosted on an on-site laptop, syncing back to the cloud PostgreSQL database intermittently when the internet connection stabilizes.

### 5.7. Strict Database Schema Isolation & Concurrency Control
- **Problem:** Database tables must explicitly prevent double-allocation errors through structural mechanics, rather than relying solely on software code lines.
- **Best Practice Solution (Explicit Row-Level Locking):**
  - When the Go service needs to finalize the booking after confirming the Redis hold layer, execute a strict explicit row-level lock statement (e.g., `SELECT ... FOR UPDATE` or strict isolated transaction state check) to prevent race conditions across parallel worker threads.


### 5.8. Audit Trail & Financial Double-Entry Ledger
- **Problem:** Mismatches between total tickets sold, payment gateway settlements, and payouts due to race conditions or silent system failures lead to irreconcilable accounts and potential financial loss.
- **Best Practice Solution (Double-Entry Bookkeeping Ledger):**
  - Never store only a single balance column in database tables. All financial movements (payment received, payout processed, service fees, discounts, and refunds) must be recorded as immutable rows in a ledger database table (credits and debits matching to zero).
  - Every booking status modification must create a permanent audit log row detailing the user ID, timestamp, transition state, and changes made.

### 5.9. Security, Anti-Bot & Scalper Mitigation
- **Problem:** Scalper bots scrape the event website and automate ticket reservation scripts, exhausting ticket stock within seconds to resell them on secondary markets.
- **Best Practice Solution (Layered Bot Mitigation):**
  - **API Rate Limiting:** Enforce strict Redis-backed token bucket rate limits on the `/api/bookings` endpoint, restricted by IP address and authenticated User ID.
  - **CAPTCHA Verification:** Force Google reCAPTCHA v3 or Cloudflare Turnstile verification during high-demand events before allowing ticket reservation requests.
  - **WAF Security Rules:** Integrate Cloudflare WAF to inspect incoming traffic signature heuristics, dropping bad actors and automated headless browsers.

### 5.10. Voucher/Promo Code Concurrency
- **Problem:** Promoters configure a limited promotional discount code (e.g. 100 uses of "PROMO20"). Under concurrent checkout hits, 150 users check out successfully with the code, violating the promoter's budget limit.
- **Best Practice Solution (Atomic Voucher Limits):**
  - Treat promotional code quotas identical to ticket stock. Use Redis atomic counters to decrement promo code use counts (`DECRBY`).
  - Check validation (expiry date, user eligibility, event ID) before reserving the code usage.
  - Bind the promo code usage to the ticket booking session (10-minute hold) so that if the booking expires, the promo code quota increment is automatically returned back to the pool.

### 5.11. Data Privacy & UU PDP Compliance
- **Problem:** Ticketing platforms collect sensitive user information (full name, email, phone number, identity card details if needed for age verification). A database leak compromises customer privacy, exposing the platform to legal penalties under local regulations (e.g., UU PDP in Indonesia / GDPR).
- **Best Practice Solution (Field-Level Encryption at Rest):**
  - Encrypt PII fields (phone numbers, identity cards/identity numbers) in the database at rest using AES-256-GCM.
  - Access keys for decryption must be stored in secure environment variables or secret managers, separate from the primary database access credentials.
  - Implement a policy for data retention and automatic anonymization of ticket buyer data 30 days after the event concludes.


### 5.12. Tax and Local Levies Compliance
- **Problem:** Ticketing platforms must handle regional amusement/entertainment taxes (Pajak Barang dan Jasa Tertentu - PBJT) and national Value Added Tax (PPN) correctly to avoid tax audits and penalties.
- **Best Practice Solution (Dynamic Tax Engine):**
  - **PBJT Rate Capping (UU HKPD No. 1/2022):** Under the Law on Financial Relations between the Central Government and Regional Governments (UU HKPD), local entertainment tax (PBJT for arts and entertainment) is capped at a maximum of **10%** for standard performances/exhibitions/concerts.
  - The pricing engine must calculate taxes dynamically based on the venue's local regional regulations (Perda) and register separate line items in invoice and ledger structures. Platform fees remain subject to standard 11% PPN.


### 5.13. Anti-Scalping Identity Binding & Secure Ticket Transfer Flow
- **Problem:** Scalper syndicates purchase tickets in bulk to resell at inflated prices, or legitimate buyers need to transfer their ticket to a friend but want to do so securely without getting scammed or allowing double-entry.
- **Best Practice Solution (NIK Binding with Cryptographic Revocation-on-Transfer):**
  - **Identity Binding:** Require a unique name and national identity number (NIK/Passport) for every single ticket during checkout. This data is cryptographically signed and embedded in the ticket QR code.
  - **Promoter Transfer Controls:** The event settings must contain toggles: `IsTransferable (boolean)`, `TransferWindowCutoffHours (integer)` (e.g., transfers close 24 hours before the event), and `TransferFee (decimal)`.
  - **Secure Transfer Execution (Two-Factor Handshake):**
    1. **Initiation:** The original buyer initiates the transfer from their dashboard. The system generates a single-use, short-lived (e.g., 1-hour expiry) cryptographic transfer token/link.
    2. **Recipient Claim:** The recipient opens the link, logs into the platform, and inputs their own name and NIK/Passport number.
    3. **Revocation & Re-issuance:** Upon acceptance, the database cancels the original ticket (`status = TRANSFERRED`), invalidates its QR code token, and issues a **brand new ticket** with a new UUID, new QR code, and updated identity records to the recipient. The original QR code will fail verification at the gate.
    4. **Limitation:** A ticket can only be transferred once to prevent chain-reselling by scalpers.

### 5.14. Virtual Waiting Room (Traffic Queue Management)
- **Problem:** A sudden surge of 100k+ users clicking "Buy Ticket" at exactly 10:00 AM saturates Go HTTP thread pools, database connections, and API gateways, causing a total system outage.
- **Best Practice Solution (Virtual Queue):**
  - Integrate a reverse proxy/CDN level virtual waiting room (e.g. Cloudflare Waiting Room) or a lightweight backend queue middleware.
  - Users are trickled into the active ticketing checkout flow at a sustainable rate (e.g., 500 requests per second) matching the database writing performance limits.
  - Active sessions display progress updates (queue position and estimated wait time) to manage user expectations and reduce page-refresh retries.

### 5.15. Offline Scan Sync Reconciliation & Conflict Handling
- **Problem:** In a local hybrid-LAN scanning architecture, two ticket checkers scan duplicate copies of the same QR code at separate gates simultaneously while offline. Both gates show green/success.
- **Best Practice Solution (Conflict Logging & Auditing):**
  - When the scanner devices reconnect to the internet and sync their check-in logs back to the cloud PostgreSQL database, the system must perform a reconciliation check.
  - If a double check-in is detected (same ticket scanned twice with separate device IDs), the database flags the ticket under a specialized `SCAN_CONFLICT` state.
  - The reconciliation worker logs detailed audit fields: check-in timestamps, scanner device IDs, and gate names. An alert is instantly triggered on the Promoter Operations Dashboard for investigation (preventing internal staff fraud or duplicate ticket prints).

### 5.16. Automated Bulk Refund Dispatch Engine
- **Problem:** An event postponement or lineup cancellation triggers thousands of refund requests. Manually processing these through payment portals is slow, error-prone, and violates local consumer protection regulations.
- **Best Practice Solution (Asynchronous Bulk Refund Queue):**
  - Provide a command/admin utility to trigger bulk refunds for a specific event or ticket tier.
  - The system publishes refund events to a dedicated RabbitMQ/Kafka queue (`ticket.refund_requested`).
  - An asynchronous worker processes refunds in batches, executing API calls to the payment gateway (e.g., Midtrans/Xendit refund endpoints) and updating database booking ledgers.
  - This prevents HTTP request timeout bottlenecks on the admin dashboard and isolates payment gateway rate-limit handling (backing off and retrying failed refund API calls automatically).

### 5.17. Hybrid Split-Payment Routing & Fee Reconciliation
- **Problem:** When charging a combination of ticket prices (destined for organizers) and platform fees (destined for the platform operator), holding the promoter's money in the platform's corporate bank accounts violates Bank Indonesia licensing regulations, artificially inflates platform tax reports, and increases administrative auditing loads.
- **Best Practice Solution (Automated Gateway-Level API Splitting):**
  - Integrate **split payment API features** from the gateway (e.g., Xendit XenPlatform or Midtrans Split Payment).
  - During checkout, the API payload instructs the payment gateway to split the total ticket price (`TotalPrice` = `OrganizerRevenue` + `PlatformFee`):
    - `PlatformFee` is routed directly to the platform's primary merchant account at the time of transaction capture.
    - `OrganizerRevenue` is routed to the promoter's virtual sub-account/wallet, remaining locked under the T+3 post-event escrow settlement policy.
  - **Refund Policy on Fees:** Specify that platform fees are non-refundable once the booking service is executed. Event cancellations only trigger the refund of the `OrganizerRevenue` portion to buyers. If a promoter insists on full customer refunds, the platform fee portion must be contractually debited from the promoter's accumulated share or security deposit.
  - **Tax Ledger Isolation:** The database ledger logs the platform fee as platform revenue (subject to 11% PPN) and the event fee as promoter revenue (subject to local PBJT handled by the organizer), ensuring clean tax invoicing.

---

## 6. Indonesian Regulatory Compliance & Legal Requirements

Operating an electronic transactional system in Indonesia requires adherence to specific legal frameworks governed by the Ministry of Communication and Informatics (Kominfo), Bank Indonesia (BI), the Ministry of Finance, and regional governments.

### 6.1. PSE Registration (Penyelenggara Sistem Elektronik)
- **Framework:** Government Regulation No. 71/2019 (PP 71/2019) and Minister of Communication and Information Regulation No. 5/2020 (MR 5/2020).
- **Implementation Rules:**
  - Karcis must register as a Private Scope PSE (PSE Lingkup Privat) under Kominfo before launching publicly.
  - The application architecture must support data localization guidelines: transaction records, user credentials, and event data must be hosted on cloud servers located within the territory of the Republic of Indonesia (e.g., Google Cloud Jakarta Region `asia-southeast2`).

### 6.2. Personal Data Protection (UU PDP No. 27/2022)
- **Framework:** Law No. 27 of 2022 on Personal Data Protection (fully enforced post-transition period).
- **Implementation Rules:**
  - **Explicit Consent:** The frontend checkout flow must include an explicit checkbox for customers to consent to the terms of service and privacy policy regarding the processing of their data.
  - **Data Breach Notification:** The system must implement automated monitoring and alerts. In the event of a security breach compromising user data, the platform is legally obligated to report the breach to the regulatory authority (Lembaga PDP) and notify affected users within **72 hours (3 days)**.
  - **Right to Erasure:** Provide a mechanism for users to request data erasure/deletion, subject to financial/tax data retention laws.

### 6.3. Electronic Transactions & Consumer Protection (UU ITE & PP 80/2019)
- **Framework:** Law No. 11 of 2008 (UU ITE) as amended by Law No. 19 of 2016 and **Law No. 1 of 2024 (Second Amendment)**, and Government Regulation No. 80 of 2019 (PP 80/2019) on Electronic Commerce.
- **Implementation Rules:**
  - E-tickets generated by Karcis must serve as legally binding electronic contracts. The terms and conditions (T&C) must be accessible, downloadable, and clearly state refund/cancellation policies before purchase.
  - Maintain a secure customer support channel (ticketing system/WhatsApp helpline) to handle disputes or complaints, as required under Indonesian consumer protection rules (UU Perlindungan Konsumen No. 8/1999).

### 6.4. Payment Processing Licensing (PBI & OJK Guidelines)
- **Framework:** Bank Indonesia Regulation **No. 23/6/PBI/2021 on Payment Service Providers (Penyedia Jasa Pembayaran - PJP)**.
- **Implementation Rules:**
  - The platform must not directly pool or hold customer funds intended for promoters in its own corporate accounts prior to payout, to avoid triggering PJP Category 1 licensing requirements.
  - Using Xendit XenPlatform / Midtrans Split APIs ensures that the payment gateway handles clearing and settlement, directly distributing the platform share and locking the promoter's share in escrow. This keeps Karcis compliant as a technology provider rather than a financial custodian.

---

## 7. Premium UI/UX Design System Guidelines

To ensure the platform avoids the generic "AI-generated template" aesthetic (which features plain white blocks, uncurated layouts, and default system fonts), Karcis must adhere to a strict, highly polished design standard.

### 7.1. Light Mode Default with Dynamic Dark Mode Support
To ensure design cohesion when users toggle between light and dark modes, the frontend must avoid hardcoded color utility classes.

#### A. Theme Configurations
- **Default State:** The application must default to **Light Mode** on initial load.
- **Theme Manager:** Manage state using `next-themes` integrated with HeroUI's `<HeroUIProvider>` to prevent Flash of Unstyled Content (FOUC).
- **Smooth Switching:** Apply Tailwind's `transition-colors duration-300` to background and text elements to ensure a smooth transition gradient when switching.

#### B. Curated Color System (Semantic Tokens)
Rather than hardcoding hex codes, style elements using HeroUI semantic Tailwind classes (`bg-background`, `text-foreground`, `bg-content1`, `border-divider`):

| Token | Light Mode Value (Default) | Dark Mode Value | Usage |
|---|---|---|---|
| `background` | Slate-50 (`#f8fafc`) | Zinc-950 (`#09090b`) | App canvas background |
| `foreground` | Slate-900 (`#0f172a`) | Zinc-50 (`#fafafa`) | Primary body text |
| `content1` | White (`#ffffff`) | Zinc-900 (`#18181b`) | Card containers / ticket stubs |
| `content2` | Slate-100 (`#f1f5f9`) | Zinc-800 (`#27272a`) | Form input fields / disabled states |
| `primary` | Violet-600 (`#7c3aed`) | Violet-500 (`#8b5cf6`) | Call-to-action buttons |
| `success` | Emerald-600 (`#059669`) | Emerald-400 (`#34d399`) | Successful scan modals |

#### C. Glassmorphism Adaptation
Glassmorphic panels (headers, mobile overlays) must adapt dynamically to prevent invisible text or excessive glare:
- **Light Mode:** `bg-white/70 backdrop-blur-md border-slate-200/50`
- **Dark Mode:** `bg-zinc-950/70 backdrop-blur-md border-zinc-800/50`

### 7.2. Skeletal & Loading States (Optimistic UI)
- **Zero Blocking Spinners:** Never overlay a full-page loading spinner. Use skeleton loaders modeled exactly after the ticket cards' layouts, animated with a subtle pulse.
- **Micro-interactions:** Buttons must use scale-down animations on click (`scale-95`) and slight upward shifts on hover (`translate-y-[-2px]`) to feel responsive and alive.

### 7.3. Interactive Ticket Checkout UI
- **Perforated Ticket Container:** The checkout page should display ticket selections inside a component that visually mimics a physical ticket stub (e.g., using Tailwind radial masks for punch-out holes and dashed borders for tear-off stubs).
- **Active State Transformations:** Selecting a ticket quantity should dynamically animate the visual stub (e.g., expanding sections downwards to reveal guest forms with a smooth spring transition using Framer Motion).
- **Urgency Display:** The checkout countdown timer must be represented as a clean, glowing progress ring that smoothly transitions from green to warning amber and alert red as the 10-minute lease window decreases.

### 7.4. Gate Scanner Operations UX
- **High-contrast Scanning Interface:** The scanner portal must utilize a high-contrast dark layout with oversized, colored status modal popups (Green for check-in success, red/yellow for duplicate warnings) that cover 80% of the screen.
- **Haptic/Audio Feedback Integration:** Use browser Web Audio APIs and Device Haptic APIs to trigger short vibratory bursts and confirmation chimes on successful scans, allowing gate staff to process tickets without continuously checking the screen.

---

## 8. Out of Scope for Phase 1
- **Dynamic Seating Maps:** Customers will purchase General Admission/seated categories without individual seat row/number selections.
- **Secondary Ticket Resale Marketplace:** P2P ticket trading/transfers will not be supported in Phase 1.
- **Apple & Google Wallet Integration:** Defer native PKPass and Google Wallet API integration to Phase 2; prioritize WhatsApp and PDF screenshot guides for offline use.
- **Multilingual Support:** Single language (English or Indonesian) support initially.
