# Company Fun Day & Game Night — IBM BAW Approval Process

An end-to-end request and approval process built in **IBM Business Automation Workflow (BAW)**. This solution automates the journey of an employee's event idea—handling initial submission, cost calculations, dynamic safety routing, parallel multi-department reviews, high-cost executive escalations, and post-approval bookings.

---

## 📌 Project Overview

When an employee submits a request to organize a company event, the system automatically evaluates process variables (such as attendee count, safety risks, and overall budget) to determine routing logic dynamically.

### Core Features

* **Dynamic Safety Evaluation:** Automatically routes requests to Safety Review if the event is held outdoors, expects over 100 attendees, or includes mechanical rides.


* **Parallel Departmental Review:** Finance and Event Operations review requests concurrently alongside Safety (when triggered).


* **Executive Escalation:** Automatically routes requests with a total estimated budget exceeding $100,000 to the Department Director for sign-off.


* **Rework & Return Loops:** Allows reviewers to send requests back to the employee for modification with reviewer feedback.


* **Fulfillment Routing:** Uses inclusive gateways to trigger parallel booking sub-tasks (Food, Activity, Transport) based on request requirements.



---

## 🏗️ Technical Architecture

### Platform & Build Metadata

* **Platform:** IBM Business Process Manager 8.6.3 (Fix Pack 21030 / BAW)


* **Process App Name:** Company Fun Day & Game Night (Acronym: `MMA`)


* **Target Environment:** `BAW_tWAS`

* **Toolkit Dependencies:** System Data (8.6.0.0), UI Toolkit (8.6.0.0)



### Process Model (BPD Structure)

The solution consists of **1 Business Process Definition (BPD)** structured across **7 swimlanes**:

1. **Employee:** Submits initial request and updates reworked submissions.


2. **System:** Handles logic scripts, reset tasks, and decision aggregations.


3. **Department Manager:** Performs initial review.


4. **Finance:** Parallel financial review.


5. **Event Operations:** Parallel operational review.


6. **Safety:** Conditional parallel safety review.


7. **Department Director:** High-cost approval (> $100k).



---

## 🗂️ Data Model (Business Objects)

The data model is structured around a composite root business object (`FunDayRequest`) and supporting child objects:

| Business Object | Key Fields / Purpose |
| --- | --- |
| **`FunDayRequest`** | Root object containing all sub-objects (`eventDetails`, `food`, `activities`, `transport`, `budget`).

 |
| **`EventDetails`** | Event name, type (Indoor/Outdoor), date, attendees, duration, location, backup location, staff required.

 |
| **`FoodDetails`** | Food required (Yes/No), package (Basic/Standard/Premium), price per person, vegetarian count, estimated cost.

 |
| **`ActivityDetails`** | Board games, karaoke, football tournament, VR tournament, mechanical ride, competition mode settings, prize budget, activity cost.

 |
| **`TransportDetails`** | Transport required (Yes/No), pickup location, bus capacity, bus count, cost per bus, total transport cost.

 |
| **`BudgetSummary`** | Itemized costs (food, activities, prizes, transport), subtotal, service fee, total estimated cost, high-cost justification.

 |
| **`ApprovalDecision`** | Shared object across review steps storing `reviewerRole`, `decision` (Approve/Reject/Return), and `comments`.

 |

---

## 💻 User Interface (Client-Side Human Services)

The UI is built using **8 Client-Side Human Services (CSHS)** and **6 custom Coach Views** with dynamic UI visibility rules:

* **Coach Views:** `CVEventDetails`, `CVFoodDetails`, `CVGamesActivities`, `CVTransportation`, `CVBudgetSummary`, `CVApprovalDecision`.


* **Dynamic UI Behavior:**
* Backup location field dynamically appears only when Event Type is set to **Outdoor**.


* Package details expand automatically when **Food Required** or **Transport Required** is toggled to **Yes**.


* High Cost Justification input displays when budget rules trigger the high-cost flag.





### Human Service Inventory

1. `CSHS_CreateFunDayRequest`: Request creation form for employees.


2. `CSHS_ManagerReview`: Department Manager review screen.


3. `CSHS_FinanceReview`: Finance review screen.


4. `CSHS_EventOpsReview`: Event Operations review screen.


5. `CSHS_SafetyReview`: Safety review screen.


6. `CSHS_DirectorApproval`: High-cost approval screen for Directors.


7. `CSHS_OutcomeNotice`: Rejection / outcome notification screen.


8. `CSHS_EventConfirmation`: Final confirmation display for approved events.



---

## ⚙️ Process Flow & Decision Rules

```
[Employee Submission] ──> [Calculate Request Script] ──> [Manager Review]
                                                                │
                            ┌───────────────────────────────────┴──────────────────────────────────┐
                            ▼ (Approved)                                                           ▼ (Returned/Rejected)
                [Parallel Reviews Gateways]                                                  [Rework / Outcome Notice]
           ┌────────────────┼────────────────┐
           ▼                ▼                ▼ (If Safety Required)
   [Finance Review]   [Event Ops]     [Safety Review]
           │                │                │
           └────────────────┼────────────────┘
                            ▼
              [Evaluate Review Results Script]
                            │
               ┌────────────┴────────────┐
               ▼ (Approved)              ▼ (Rework / Rejected)
       [Over 100k Check]           [Rework / Outcome Notice]
         │          │
         │ (> 100k) └─────────────┐ (<= 100k)
         ▼                        │
 [Director Approval]              │
         │                        │
         └──────────┬─────────────┘
                    ▼
          [Inclusive Bookings] ──> [Event Confirmation] ──> (Success End)

```

1. **Calculate Request Script:** Evaluates input variables to set routing flags (`safetyRequired`, `directorRequired`, `needFoodBooking`, `needActivityBooking`, `needTransportBooking`).


2. **Evaluate Review Results Script:** Combines parallel review decisions:
* Any **Reject** decision sets the overall outcome to `Rejected`.


* Any **Return for Modification** decision sets the overall outcome to `Rework`.


* Unanimous approvals set the outcome to `Approved`.





---

## 🧪 Test Scenarios Covered

1. **Standard Indoor Event:** Budget under $100k, indoor location, < 100 attendees → Bypasses Safety and Director approvals.


2. **Outdoor / High-Risk Event:** Outdoor location, > 100 attendees, or mechanical ride selected → Successfully triggers Safety Review lane.


3. **High-Budget Escalation:** Budget > $100k → Triggers Director Approval step with high-cost justification visible.


4. **Rework Loops:** Manager, Finance, or Safety returns request → Successfully routes back to Employee with reviewer feedback.


5. **Draft Operations:** Save draft functionality verifies data persistence before submission.
