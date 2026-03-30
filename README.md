# 🏦 Ram Yellow Bank Agent

> A GenAI-powered Banking Agent built on [Yellow.ai](https://yellow.ai) that enables secure user authentication and structured loan detail retrieval through multi-step API workflows.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Workflow Architecture](#workflow-architecture)
- [Token Optimization](#token-optimization)
- [Mock API Setup](#mock-api-setup)
- [Edge Cases Handled](#edge-cases-handled)
- [CSAT Feedback System](#csat-feedback-system)
- [Bot Language Restriction](#bot-language-restriction)
- [How to Run](#how-to-run)
- [Screenshots](#screenshots)

---

## 📖 Overview

This project implements a **GenAI Banking Agent** for **Yellow Bank** using the Yellow.ai platform.  
The agent guides users through a complete loan detail retrieval journey — from identity verification to structured loan data display — with token-efficient API handling and feedback collection.

**Key design goals:**
- Secure multi-step authentication (Phone + DOB + OTP)
- Token-optimized API response handling (Projection Method)
- Dynamic Rich Media (DRM) card rendering
- Clean CSAT feedback collection
- English-only conversation enforcement

---

## 🛠 Tech Stack

| Tool | Purpose |
|------|---------|
| [Yellow.ai](https://cloud.yellow.ai) | Bot platform & workflow builder |
| [Beeceptor](https://beeceptor.com) | Mock API hosting |
| Yellow.ai Function Nodes | Token optimization / projection logic |
| Yellow.ai DRM Cards | Interactive UI rendering |

---

## ✨ Features

### 1. 🔐 User Authentication
- Collects **Registered Phone Number**
- Collects **Date of Birth (DOB)**
- Validates format before proceeding to OTP step

### 2. 📲 OTP Verification
- Triggers `triggerOTP` workflow after Phone + DOB collection
- Mock API returns one of four predetermined OTPs: `1234`, `5678`, `7889`, `1209`
- Agent asks user to enter OTP and verifies it before continuing
- Handles incorrect OTP with retry and failure messaging

### 3. 💳 Loan Account Retrieval
- Triggers `getLoanAccounts` workflow
- Displays loan accounts as **interactive DRM cards**, each showing:
  - Loan Account ID
  - Loan Type
  - Tenure
- User selects an account by clicking the **Select** button on the card
- Button automatically sends `loan_account_id` as message value

### 4. 📋 Loan Details Retrieval
- Triggers `getLoanDetails` workflow with the selected `loan_account_id`
- Displays all 5 required fields via **DRM with Quick Replies**:
  - Tenure
  - Interest Rate
  - Principal Pending
  - Interest Pending
  - Nominee
- Includes **"Rate our chat"** button to trigger CSAT flow

### 5. ⭐ CSAT Feedback System
- Triggered when user clicks **"Rate our chat"**
- Shows rating as **Quick Reply buttons**: `Good` / `Average` / `Bad`
- Collects optional text feedback
- Confirms with a thank you message and ends cleanly

---

## 🔄 Workflow Architecture

```
User Intent (Loan Details)
        │
        ▼
┌─────────────────────┐
│  Phone Number Input │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   DOB Input         │
└────────┬────────────┘
         │
         ▼
┌─────────────────────────────┐
│  Workflow 1: triggerOTP     │
│  → Mock API → OTP value     │
│  → Agent asks user for OTP  │
│  → Verifies OTP             │
└────────┬────────────────────┘
         │
         ▼
┌──────────────────────────────────┐
│  Workflow 2: getLoanAccounts     │
│  → Mock API (15+ fields JSON)    │
│  → Function Node (Projection)    │
│  → DRM Cards (3 fields only)     │
│  → User selects account          │
└────────┬─────────────────────────┘
         │
         ▼
┌──────────────────────────────────┐
│  Workflow 3: getLoanDetails      │
│  → Mock API → 5 fields           │
│  → DRM Quick Reply display       │
│  → "Rate our chat" button        │
└────────┬─────────────────────────┘
         │
         ▼
┌──────────────────────┐
│  CSAT Flow           │
│  → Good/Average/Bad  │
│  → Optional feedback │
│  → Thank you + End   │
└──────────────────────┘
```

---

## ⚡ Token Optimization

The `getLoanAccounts` mock API intentionally returns a **large JSON with 15+ fields** per loan object (internal bank codes, audit dates, branch codes, etc.).

To prevent the LLM from processing unnecessary tokens and to avoid hallucinations, a **Function Node (Projection Method)** is used as a middleware layer:

```javascript
return new Promise(resolve => {
  let raw = data.variables.raw_loans_response;
  let loans = (raw && raw.loans) ? raw.loans : [];

  const filtered = loans.map(acc => ({
    loan_account_id: acc.loan_account_id,
    loan_type: acc.loan_type,
    tenure: acc.tenure
  }));

  resolve({ filtered_loans: filtered });
});
```

**Result:** Only 3 fields (`loan_account_id`, `loan_type`, `tenure`) are passed to the LLM — all other 12+ fields are stripped before reaching the agent.

---

## 🌐 Mock API Setup (Beeceptor)

### Endpoint 1 — OTP API
```
GET https://<your-endpoint>.free.beeceptor.com/otp
Response: { "otp": "1234" }
```

### Endpoint 2 — Loan Accounts API
```
GET https://<your-endpoint>.free.beeceptor.com/loans
Response: { "loans": [ { "loan_account_id": "LN-1001", "loan_type": "Home Loan", "tenure": "20 years", ...15+ fields } ] }
```

### Endpoint 3 — Loan Details API
```
GET https://<your-endpoint>.free.beeceptor.com/loan-details
Response:
{
  "tenure": "20 years",
  "interest_rate": "8.5%",
  "principal_pending": "1800000",
  "interest_pending": "320000",
  "nominee": "Priya Sharma"
}
```

---

## 🔀 Edge Cases Handled

| Scenario | Bot Behaviour |
|----------|--------------|
| Wrong OTP entered | Asks user to re-enter, retries up to limit |
| API failure | Shows friendly error message, suggests retry |
| User says "that's my old number" | Clears Phone + DOB slots, retains intent, restarts collection |
| Invalid phone number format | Prompts user to enter a valid 10-digit number |

---

## 🌍 Bot Language Restriction

The agent is restricted to **English only**.

If a user types in any other language (Hindi, Telugu, Spanish, etc.), the bot responds:

> *"I'm sorry, I can only assist you in English. Please type your query in English."*

This is enforced via:
- System prompt instruction
- Yellow.ai supported language configuration (English only)
- Global fallback message

---

## 🚀 How to Run

1. Log in to [cloud.yellow.ai](https://cloud.yellow.ai)
2. Open the bot: **Ram Yellow Bank Agent**
3. Go to **Studio → Preview** or open the live PWA link
4. Start conversation: type `"I want to check my loan details"`
5. Follow the prompts: Phone → DOB → OTP → Select Loan → View Details → Rate

---

## 📸 Screenshots

| Step | Description |
|------|-------------|
| Auth Flow | Phone number and DOB collection |
| OTP Verification | OTP entry and validation |
| Loan Cards | Interactive DRM cards with Select button |
| Loan Details | All 5 fields displayed with Rate button |
| CSAT | Good / Average / Bad quick reply rating |

---

## 👤 Author

**Ram**  
MCA Student  
Built as part of Yellow.ai GenAI Banking Agent Assignment

---

## 📄 License

This project is built for educational/assignment purposes using Yellow.ai platform.
