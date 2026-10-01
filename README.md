# WORKNOON — AI-Powered Customer Support Refund System

A containerized full-stack assessment demo for processing e-commerce refund requests with server-side policy enforcement, AI decision support, audit persistence, and a support dashboard.

## Architecture

React/Vite frontend → Express REST API → policy engine + JSON data store → OpenAI Responses API (optional AI reasoning) → persisted refund audit records.

The synthetic data store contains 15 customer profiles and order histories. JSON storage was chosen as an explicit lightweight option permitted by the assessment and keeps the local container setup simple.

## Run locally

### Docker (recommended)

1. Copy `backend/.env.example` to `backend/.env`.
2. Add an OpenAI API key if AI reasoning is required:

```env
OPENAI_API_KEY=your_key_here
OPENAI_MODEL=gpt-5.6-luna
```

3. Run:

```bash
docker compose up --build
```

Open `http://localhost:5173`.

The API health endpoint is `http://localhost:5000/api/health`.

### Development without Docker

Backend:

```bash
cd backend
npm install
npm run dev
```

Frontend:

```bash
cd frontend
npm install
npm run dev
```

## AI integration

The backend uses the OpenAI Responses API. The policy engine evaluates trusted customer/order data first. The AI receives the trusted facts, policy rules, customer message as untrusted data, and the deterministic policy decision. The AI can explain the result but cannot override the server-side decision.

If no API key is supplied, the application remains functional using deterministic policy decisions and clearly records that AI reasoning was unavailable.

## Security / prompt injection safeguards

- Customer messages are treated as untrusted input.
- Basic prompt-injection and policy-bypass patterns are detected.
- Policy checks run server-side before AI reasoning.
- The AI is explicitly instructed that customer text cannot override policy.
- AI output is explanatory; the final decision remains the policy-engine result.
- Request bodies have a size limit and required fields are validated.
- API keys are provided through environment variables and are not stored in source control.

## Policy assumptions

- Refund window: 30 days.
- Refunds above $500 require human review.
- Final-sale items are denied.
- Damaged/incorrect items within the window and below the threshold can be approved.
- Ambiguous cases are escalated.

## Demo flow

1. Open Customer Request.
2. Choose a customer/order.
3. Submit a damaged or incorrect-item request and show APPROVED.
4. Submit an older/final-sale/high-value case and show DENIED or ESCALATED.
5. Open Support Dashboard and show the persisted audit record and AI/policy reasoning.

## Trade-off

The assessment explicitly permits JSON-based storage. It was selected over a database server to keep the single-command containerized demo fast, portable, and easy for reviewers to inspect.
