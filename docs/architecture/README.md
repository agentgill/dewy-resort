# Architecture Diagrams

This directory contains visual documentation of the Dewy Resort Hotel's Compositional MCP Architecture using Workato as the central orchestration layer.

## System Architecture

### system-architecture.png

**Dewy Resort Hotel - System Architecture & Integration Landscape**

This diagram illustrates the complete system architecture showing:

- **Hotel Application Layer**
  - Next.js App (Frontend & API Routes)
  - SQLite Database (Local data storage)
  - Home Assistant (IoT device control)

- **Workato Integration Hub** (Central Orchestration Layer)
  - Compositional MCP Architecture
  - Multiple MCP servers organized by user persona and "jobs to be done"

- **Persona-Based MCP Servers**
  - **Guest MCP Server**: Guest-facing workflows and self-service
    - Orchestrators: `check_in_guest`, `checkout_guest`, `service_request`
    - Atomic Skills: `search_contact`, `search_booking`, `upsert_case`

  - **Staff MCP Server**: Staff operations and management
    - Orchestrators: `maintenance_request`, `assign_room`, `update_case_status`
    - Atomic Skills: `create_contact`, `search_room`, `update_room_status`

- **API Platform Layer**
  - REST endpoints backing MCP tools
  - Recipe orchestration
  - Multi-system workflows

- **External Systems**
  - **Salesforce**: CRM + Analytics (service requests, guest contacts, bookings & rooms, reports & dashboards)
  - **Twilio**: SMS Communications (booking confirmations, status updates, check-in reminders)
  - **Stripe**: Payment Processing (guest checkout, refunds, payment receipts)

**Key Architectural Principle:**
> "All data movement orchestrated through Workato - No direct system integrations"

**Workshop Concept: Compositional MCP Architecture**
- **Orchestrators (High-level)**: Encode common workflows and business rules for happy-path scenarios. Fast, validated, convenient for common use cases.
- **Atomic Skills (Building blocks)**: Low-level operations that AI agents can intelligently compose at runtime to handle edge cases and dynamic scenarios.

**Example**: When a guest says "I need towels in room 101" and the contact doesn't exist:
- Orchestrator would fail (requires existing contact)
- AI agent uses atomic skills: `create_contact` → `upsert_case` with proper approval authority

---

## Orchestrator Flow Diagrams

These diagrams show detailed sequence flows for key orchestrator recipes, demonstrating how MCP tools call orchestrators, which validate prerequisites and execute atomic Salesforce operations.

### guest-checkin-flow.png

**Guest Check-In - Multi-object state transition orchestrator**

**Overview:**
- **Execution Time**: < 3 seconds
- **API Operations**: 6 API calls (3 reads + 3 updates)
- **Orchestrator Type**: Multi-object state transition

**Sequence Flow:**

1. **MCP Tool** - `POST /check-in-guest`
   - Input: `{guest_email, check_in_date}`

2. **Orchestrator** - Validate input & prerequisites

3. **Salesforce Atomic Operations** (parallel execution):
   - Search Contact by Email
     - ✓ Guest exists? (Contact.id)
     - ✗ 404 - Guest not found
   - Search Booking (Reserved status)
     - ✓ Reservation exists? (Booking.id)
     - ✗ 404 - No reservation | 409 - Multiple bookings
   - Check Room Status
     - ✓ Room vacant?
     - ✗ 409 - Room not available

4. **Orchestrator** - Execute State Transitions (sequential, dependency order):
   1. Booking: Reserved → Checked In
   2. Room: Vacant → Occupied
   3. Opportunity: Booking Confirmed → Checked In

5. **MCP Tool** - Return Success
   - ✓ booking_id, room_number, check_in_date

**Workshop Teaching Points:**
- Prerequisites: Guest + Reservation + Vacant Room required
- State transitions in dependency order
- Idempotent via booking_id check
- No auto-creation - requires human intervention for edge cases

---

### guest-checkout-flow.png

**Guest Checkout - Complete checkout workflow with payment processing**

**Overview:**
- Validates guest booking is in "Checked In" status
- Processes payment via Stripe integration
- Updates room status to "Cleaning"
- Updates booking status to "Checked Out"
- Updates opportunity stage to "Closed Won"

**Key Features:**
- Payment validation and processing
- Multi-system coordination (Salesforce + Stripe)
- Room turnover workflow initiation
- Error handling for payment failures

**Workshop Teaching Points:**
- Multi-system orchestration (Salesforce + Stripe)
- Payment idempotency via Stripe payment intent
- Graceful failure handling (payment declined scenarios)
- Room status lifecycle management

---

### guest-service-request-flow.png

**Guest Service Request - Service request submission workflow**

**Overview:**
- Guest submits service request via MCP tool
- Validates guest booking exists
- Creates Case in Salesforce (Service Request type)
- Links case to booking and room
- Optionally sends SMS confirmation via Twilio

**Key Features:**
- Case creation with proper linkage (Booking, Room, Contact)
- External ID for idempotency
- SMS notification integration (optional)
- Priority and status assignment

**Workshop Teaching Points:**
- Case creation patterns in Salesforce
- External ID usage for idempotency
- Multi-object relationships (Case → Booking → Room)
- Optional SMS notification flow

---

### maintenance-request-flow.png

**Maintenance Request - Staff maintenance workflow**

**Overview:**
- Staff member submits maintenance request via MCP tool
- Validates room exists and maintenance is needed
- Creates Case in Salesforce (Facilities type)
- Assigns to appropriate maintenance team
- Updates room status to "Maintenance"
- Optionally notifies vendor via Twilio

**Key Features:**
- Staff-specific workflow with elevated permissions
- Room status management during maintenance
- Vendor assignment and notification
- Case routing logic

**Workshop Teaching Points:**
- Permission-based workflows (Staff vs Guest)
- Room status transitions (Occupied/Vacant → Maintenance → Cleaning → Vacant)
- Vendor management and notification
- Case assignment automation

---

## Usage in Documentation

These diagrams are embedded in:

- **README.md** - Main project overview (system architecture + guest check-in example)
- **docs/WORKATO_SETUP.md** - Workato setup guide (all orchestrator flows)
- **Workshop presentations** - Visual aids for teaching MCP architecture concepts

---

## Diagram Maintenance

These diagrams were created to visualize the Compositional MCP Architecture pattern. When updating:

1. **System Architecture** - Update when adding new MCP servers, external systems, or changing the integration hub structure
2. **Orchestrator Flows** - Update when modifying recipe logic, adding new state transitions, or changing API operations

The diagrams serve as both documentation and workshop teaching aids, demonstrating:
- How orchestrators encode business rules for common scenarios
- How atomic skills enable AI agents to handle edge cases
- Why zero direct integrations (all through Workato) matters
- The performance characteristics of orchestrated workflows

---

## Related Documentation

- [Main README](../../README.md) - Project overview with architecture section
- [Workato Setup Guide](../WORKATO_SETUP.md) - Recipe deployment and MCP architecture details
- [Salesforce Setup Guide](../SALESFORCE_SETUP.md) - Data layer setup (prerequisite)
