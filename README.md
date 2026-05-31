🦷 Dental Claims Intelligence API

AI-powered dental insurance claims analysis platform that detects fraud, billing anomalies, and medical necessity issues — helping insurers make faster, smarter decisions.

Built with Node.js · Express · MongoDB · Anthropic Claude AI

📌 Overview
Dental insurance fraud costs the industry over $12 billion every year. A large portion comes from:

Upcoding — billing for more expensive procedures than performed
Phantom billing — charging for procedures that never happened
Unbundling — splitting a single procedure into multiple codes to inflate payment
Duplicate claims — submitting the same claim multiple times

Traditional claims review is slow, manual, and inconsistent. Human reviewers can only catch a fraction of fraudulent submissions before payment is released.
This API solves that. It plugs into any insurance workflow and instantly runs every incoming claim through a two-stage AI pipeline — combining fast rule-based pre-screening with deep Claude AI analysis — to produce a risk score, flag suspicious patterns, and recommend approve / deny / review before a single dollar is paid out.

✨ Features
#FeatureDescription🤖AI Claim AnalysisClaude AI evaluates each claim for fraud, upcoding, and billing errors⚡Rule-Based Pre-screeningInstant checks for duplicate CDT codes, missing X-rays, and abnormal fees📊Risk ScoringEvery claim gets a 0–100 risk score with low / medium / high classification🔄Auto Status UpdatesHigh-risk claims are auto-flagged; clean claims are auto-approved🦷CDT Code ValidationEnforces valid D0000–D9999 dental billing code format on every procedure📄Paginated FilteringFilter claims by status, provider, or insurer with full pagination📈Stats DashboardPortfolio-level overview — claims by status, total amounts, average risk score🔒Production ReadyHelmet security headers, CORS, Morgan logging, centralized error handling

🏗️ Architecture
dental-claims-api/
│
├── src/
│   ├── app.js                      # Express app entry point & middleware stack
│   │
│   ├── config/
│   │   └── db.js                   # MongoDB connection with error handling
│   │
│   ├── controllers/
│   │   └── claimsController.js     # All route handlers & core business logic
│   │
│   ├── middleware/
│   │   └── errorHandler.js         # Global error handler & 404 catcher
│   │
│   ├── models/
│   │   └── Claim.js                # Mongoose schema, validation rules & DB indexes
│   │
│   ├── routes/
│   │   └── claims.js               # Express router — maps URLs to controllers
│   │
│   └── services/
│       └── aiService.js            # Claude API integration + rule-based pre-screening
│
└── tests/
    └── claims.test.js              # 10 integration tests (Jest + Supertest)

🧠 AI Pipeline
Every submitted claim flows through a two-stage analysis pipeline before a decision is made:
┌─────────────────────────────────────────────────┐
│                 CLAIM SUBMITTED                  │
│         POST /api/claims/:id/analyze             │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│           STAGE 1 — PRE-SCREENING               │
│              (Instant · Rule-based)              │
│                                                  │
│  ✓ Duplicate CDT codes?                          │
│  ✓ Single procedure fee > $5,000?                │
│  ✓ Claim > $10,000 without X-ray?                │
│  ✓ Restorative work with no diagnosis?           │
└─────────────────────┬───────────────────────────┘
                      │
                      │  flags passed forward
                      ▼
┌─────────────────────────────────────────────────┐
│           STAGE 2 — CLAUDE AI ANALYSIS          │
│           (Deep · Semantic · Contextual)         │
│                                                  │
│  ✓ Are CDT codes appropriate for the diagnosis?  │
│  ✓ Are fees reasonable for this procedure?       │
│  ✓ Any upcoding or unbundling patterns?          │
│  ✓ Is medical necessity supported?               │
│  ✓ Missing documentation red flags?              │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│                  DECISION                        │
│                                                  │
│  riskScore  : 0 – 100                            │
│  riskLevel  : low │ medium │ high                │
│  confidence : 0 – 100%                           │
│                                                  │
│  low + approve   →  status: approved  ✅          │
│  high risk       →  status: flagged   🚩          │
│  anything else   →  status: under_review 🔍       │
└─────────────────────────────────────────────────┘

🚀 Getting Started
Step 1 — Clone the repo
bashgit clone https://github.com/kainatfatima/dental-claims-api.git
cd dental-claims-api
Step 2 — Install dependencies
bashnpm install
Step 3 — Configure environment
bashcp .env.example .env
Open .env and fill in your values:
envPORT=3000
MONGODB_URI=mongodb://localhost:27017/dental-claims
ANTHROPIC_API_KEY=your_api_key_here
NODE_ENV=development
Get your free Anthropic API key at → https://console.anthropic.com
Step 4 — Run the server
bashnpm run dev
✅ MongoDB connected: localhost
🚀 Server running on http://localhost:3000
🦷 Claims API:   http://localhost:3000/api/claims

📡 API Reference
Base URL
http://localhost:3000
Endpoints
MethodEndpointDescriptionGET/healthHealth check — confirms server is runningPOST/api/claimsSubmit a new dental claimGET/api/claimsList all claims (paginated, filterable)GET/api/claims/:idGet a single claim by IDPATCH/api/claims/:id/statusManually update claim statusPOST/api/claims/:id/analyzeRun full AI analysis on a claimGET/api/claims/stats/overviewPortfolio-level statistics dashboard
Query Parameters for GET /api/claims
ParameterTypeDescriptionExamplestatusstringFilter by claim status?status=flaggedproviderIdstringFilter by dental provider?providerId=PROV-001insurerIdstringFilter by insurer?insurerId=INS-AETNApagenumberPage number (default: 1)?page=2limitnumberResults per page (default: 20)?limit=10
Claim Status Values
StatusMeaningpendingSubmitted, not yet analyzedapprovedAI cleared — low riskunder_reviewMedium risk — needs human reviewflaggedHigh risk — potential fraud detecteddeniedManually denied by reviewer

📋 Example Walkthrough
1. Submit a Claim
bashcurl -X POST http://localhost:3000/api/claims \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "Jane Doe",
    "patientDOB": "1985-06-15",
    "providerId": "PROV-001",
    "providerName": "Bright Smiles Dental",
    "insurerId": "INS-AETNA",
    "diagnosis": "Dental caries on lower molar",
    "xrayIncluded": true,
    "procedures": [
      {
        "cdtCode": "D0120",
        "description": "Periodic oral evaluation",
        "fee": 75
      },
      {
        "cdtCode": "D2140",
        "description": "Amalgam restoration, 1 surface",
        "toothNumber": "14",
        "surface": "O",
        "fee": 180
      }
    ],
    "totalAmount": 255
  }'
json{
  "success": true,
  "message": "Claim submitted successfully",
  "data": {
    "claimId": "CLM-A1B2C3D4",
    "status": "pending",
    "patientName": "Jane Doe",
    "totalAmount": 255,
    "createdAt": "2026-05-31T10:00:00.000Z"
  }
}

2. Run AI Analysis
bashcurl -X POST http://localhost:3000/api/claims/CLM-A1B2C3D4/analyze
json{
  "success": true,
  "message": "Claim analysis complete",
  "data": {
    "claimId": "CLM-A1B2C3D4",
    "status": "approved",
    "aiAnalysis": {
      "riskScore": 12,
      "riskLevel": "low",
      "flags": [],
      "recommendation": "approve",
      "confidence": 91,
      "analyzedAt": "2026-05-31T10:00:05.000Z"
    },
    "summary": "Routine claim with standard CDT codes and reasonable fees. No billing anomalies detected.",
    "preScreenWarnings": []
  }
}

3. Check Portfolio Stats
bashcurl http://localhost:3000/api/claims/stats/overview
json{
  "success": true,
  "data": {
    "byStatus": [
      { "_id": "approved",     "count": 42, "totalAmount": 18500 },
      { "_id": "under_review", "count": 11, "totalAmount": 9800  },
      { "_id": "flagged",      "count": 7,  "totalAmount": 43200 },
      { "_id": "pending",      "count": 15, "totalAmount": 6750  }
    ],
    "averageRiskScore": "34.2"
  }
}

🧪 Tests
bashnpm test
  Health Check
    ✓ GET /health returns status ok

  POST /api/claims
    ✓ creates a claim with a generated claimId
    ✓ returns 400 on validation error
    ✓ returns 400 for invalid CDT code format

  GET /api/claims
    ✓ returns empty list when no claims exist
    ✓ returns claims list with pagination metadata
    ✓ filters by status

  GET /api/claims/:id
    ✓ retrieves a claim by claimId
    ✓ returns 404 for non-existent claim

  PATCH /api/claims/:id/status
    ✓ updates status and notes
    ✓ rejects invalid status

  GET /api/claims/stats/overview
    ✓ returns stats breakdown by status

  404 handler
    ✓ returns 404 for unknown routes

Tests: 10 passed ✅  |  Time: ~1s

🛠️ Tech Stack
LayerTechnologyWhyRuntimeNode.js 18+Fast, non-blocking I/O — ideal for API servicesFrameworkExpress 4Minimal, flexible HTTP routingDatabaseMongoDB + MongooseFlexible schema for varied claim structuresAI EngineAnthropic Claude (Sonnet)State-of-the-art reasoning for fraud detectionSecurityHelmet + CORSIndustry-standard HTTP hardeningTestingJest + SupertestFull integration test coverageLoggingMorganStructured HTTP request loggingDev ToolsNodemonAuto-reload during development

🔮 Roadmap

 JWT authentication with insurer / provider roles
 Batch claim submission endpoint
 Webhook notifications on status change
 Docker + docker-compose support
 Swagger / OpenAPI documentation
 Historical fraud trend analysis per provider


👩‍💻 Author
Kainat Fatima

GitHub: Kainat744
LinkedIn:🦷 Dental Claims Intelligence API

AI-powered dental insurance claims analysis platform that detects fraud, billing anomalies, and medical necessity issues — helping insurers make faster, smarter decisions.

Built with Node.js · Express · MongoDB · Anthropic Claude AI

📌 Overview
Dental insurance fraud costs the industry over $12 billion every year. A large portion comes from:

Upcoding — billing for more expensive procedures than performed
Phantom billing — charging for procedures that never happened
Unbundling — splitting a single procedure into multiple codes to inflate payment
Duplicate claims — submitting the same claim multiple times

Traditional claims review is slow, manual, and inconsistent. Human reviewers can only catch a fraction of fraudulent submissions before payment is released.
This API solves that. It plugs into any insurance workflow and instantly runs every incoming claim through a two-stage AI pipeline — combining fast rule-based pre-screening with deep Claude AI analysis — to produce a risk score, flag suspicious patterns, and recommend approve / deny / review before a single dollar is paid out.

✨ Features
#FeatureDescription🤖AI Claim AnalysisClaude AI evaluates each claim for fraud, upcoding, and billing errors⚡Rule-Based Pre-screeningInstant checks for duplicate CDT codes, missing X-rays, and abnormal fees📊Risk ScoringEvery claim gets a 0–100 risk score with low / medium / high classification🔄Auto Status UpdatesHigh-risk claims are auto-flagged; clean claims are auto-approved🦷CDT Code ValidationEnforces valid D0000–D9999 dental billing code format on every procedure📄Paginated FilteringFilter claims by status, provider, or insurer with full pagination📈Stats DashboardPortfolio-level overview — claims by status, total amounts, average risk score🔒Production ReadyHelmet security headers, CORS, Morgan logging, centralized error handling

🏗️ Architecture
dental-claims-api/
│
├── src/
│   ├── app.js                      # Express app entry point & middleware stack
│   │
│   ├── config/
│   │   └── db.js                   # MongoDB connection with error handling
│   │
│   ├── controllers/
│   │   └── claimsController.js     # All route handlers & core business logic
│   │
│   ├── middleware/
│   │   └── errorHandler.js         # Global error handler & 404 catcher
│   │
│   ├── models/
│   │   └── Claim.js                # Mongoose schema, validation rules & DB indexes
│   │
│   ├── routes/
│   │   └── claims.js               # Express router — maps URLs to controllers
│   │
│   └── services/
│       └── aiService.js            # Claude API integration + rule-based pre-screening
│
└── tests/
    └── claims.test.js              # 10 integration tests (Jest + Supertest)

🧠 AI Pipeline
Every submitted claim flows through a two-stage analysis pipeline before a decision is made:
┌─────────────────────────────────────────────────┐
│                 CLAIM SUBMITTED                  │
│         POST /api/claims/:id/analyze             │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│           STAGE 1 — PRE-SCREENING               │
│              (Instant · Rule-based)              │
│                                                  │
│  ✓ Duplicate CDT codes?                          │
│  ✓ Single procedure fee > $5,000?                │
│  ✓ Claim > $10,000 without X-ray?                │
│  ✓ Restorative work with no diagnosis?           │
└─────────────────────┬───────────────────────────┘
                      │
                      │  flags passed forward
                      ▼
┌─────────────────────────────────────────────────┐
│           STAGE 2 — CLAUDE AI ANALYSIS          │
│           (Deep · Semantic · Contextual)         │
│                                                  │
│  ✓ Are CDT codes appropriate for the diagnosis?  │
│  ✓ Are fees reasonable for this procedure?       │
│  ✓ Any upcoding or unbundling patterns?          │
│  ✓ Is medical necessity supported?               │
│  ✓ Missing documentation red flags?              │
└─────────────────────┬───────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────┐
│                  DECISION                        │
│                                                  │
│  riskScore  : 0 – 100                            │
│  riskLevel  : low │ medium │ high                │
│  confidence : 0 – 100%                           │
│                                                  │
│  low + approve   →  status: approved  ✅          │
│  high risk       →  status: flagged   🚩          │
│  anything else   →  status: under_review 🔍       │
└─────────────────────────────────────────────────┘

🚀 Getting Started
Step 1 — Clone the repo
bashgit clone https://github.com/kainatfatima/dental-claims-api.git
cd dental-claims-api
Step 2 — Install dependencies
bashnpm install
Step 3 — Configure environment
bashcp .env.example .env
Open .env and fill in your values:
envPORT=3000
MONGODB_URI=mongodb://localhost:27017/dental-claims
ANTHROPIC_API_KEY=your_api_key_here
NODE_ENV=development
Get your free Anthropic API key at → https://console.anthropic.com
Step 4 — Run the server
bashnpm run dev
✅ MongoDB connected: localhost
🚀 Server running on http://localhost:3000
🦷 Claims API:   http://localhost:3000/api/claims

📡 API Reference
Base URL
http://localhost:3000
Endpoints
MethodEndpointDescriptionGET/healthHealth check — confirms server is runningPOST/api/claimsSubmit a new dental claimGET/api/claimsList all claims (paginated, filterable)GET/api/claims/:idGet a single claim by IDPATCH/api/claims/:id/statusManually update claim statusPOST/api/claims/:id/analyzeRun full AI analysis on a claimGET/api/claims/stats/overviewPortfolio-level statistics dashboard
Query Parameters for GET /api/claims
ParameterTypeDescriptionExamplestatusstringFilter by claim status?status=flaggedproviderIdstringFilter by dental provider?providerId=PROV-001insurerIdstringFilter by insurer?insurerId=INS-AETNApagenumberPage number (default: 1)?page=2limitnumberResults per page (default: 20)?limit=10
Claim Status Values
StatusMeaningpendingSubmitted, not yet analyzedapprovedAI cleared — low riskunder_reviewMedium risk — needs human reviewflaggedHigh risk — potential fraud detecteddeniedManually denied by reviewer

📋 Example Walkthrough
1. Submit a Claim
bashcurl -X POST http://localhost:3000/api/claims \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "Jane Doe",
    "patientDOB": "1985-06-15",
    "providerId": "PROV-001",
    "providerName": "Bright Smiles Dental",
    "insurerId": "INS-AETNA",
    "diagnosis": "Dental caries on lower molar",
    "xrayIncluded": true,
    "procedures": [
      {
        "cdtCode": "D0120",
        "description": "Periodic oral evaluation",
        "fee": 75
      },
      {
        "cdtCode": "D2140",
        "description": "Amalgam restoration, 1 surface",
        "toothNumber": "14",
        "surface": "O",
        "fee": 180
      }
    ],
    "totalAmount": 255
  }'
json{
  "success": true,
  "message": "Claim submitted successfully",
  "data": {
    "claimId": "CLM-A1B2C3D4",
    "status": "pending",
    "patientName": "Jane Doe",
    "totalAmount": 255,
    "createdAt": "2026-05-31T10:00:00.000Z"
  }
}

2. Run AI Analysis
bashcurl -X POST http://localhost:3000/api/claims/CLM-A1B2C3D4/analyze
json{
  "success": true,
  "message": "Claim analysis complete",
  "data": {
    "claimId": "CLM-A1B2C3D4",
    "status": "approved",
    "aiAnalysis": {
      "riskScore": 12,
      "riskLevel": "low",
      "flags": [],
      "recommendation": "approve",
      "confidence": 91,
      "analyzedAt": "2026-05-31T10:00:05.000Z"
    },
    "summary": "Routine claim with standard CDT codes and reasonable fees. No billing anomalies detected.",
    "preScreenWarnings": []
  }
}

3. Check Portfolio Stats
bashcurl http://localhost:3000/api/claims/stats/overview
json{
  "success": true,
  "data": {
    "byStatus": [
      { "_id": "approved",     "count": 42, "totalAmount": 18500 },
      { "_id": "under_review", "count": 11, "totalAmount": 9800  },
      { "_id": "flagged",      "count": 7,  "totalAmount": 43200 },
      { "_id": "pending",      "count": 15, "totalAmount": 6750  }
    ],
    "averageRiskScore": "34.2"
  }
}

🧪 Tests
bashnpm test
  Health Check
    ✓ GET /health returns status ok

  POST /api/claims
    ✓ creates a claim with a generated claimId
    ✓ returns 400 on validation error
    ✓ returns 400 for invalid CDT code format

  GET /api/claims
    ✓ returns empty list when no claims exist
    ✓ returns claims list with pagination metadata
    ✓ filters by status

  GET /api/claims/:id
    ✓ retrieves a claim by claimId
    ✓ returns 404 for non-existent claim

  PATCH /api/claims/:id/status
    ✓ updates status and notes
    ✓ rejects invalid status

  GET /api/claims/stats/overview
    ✓ returns stats breakdown by status

  404 handler
    ✓ returns 404 for unknown routes

Tests: 10 passed ✅  |  Time: ~1s

🛠️ Tech Stack
LayerTechnologyWhyRuntimeNode.js 18+Fast, non-blocking I/O — ideal for API servicesFrameworkExpress 4Minimal, flexible HTTP routingDatabaseMongoDB + MongooseFlexible schema for varied claim structuresAI EngineAnthropic Claude (Sonnet)State-of-the-art reasoning for fraud detectionSecurityHelmet + CORSIndustry-standard HTTP hardeningTestingJest + SupertestFull integration test coverageLoggingMorganStructured HTTP request loggingDev ToolsNodemonAuto-reload during development

🔮 Roadmap

 JWT authentication with insurer / provider roles
 Batch claim submission endpoint
 Webhook notifications on status change
 Docker + docker-compose support
 Swagger / OpenAPI documentation
 Historical fraud trend analysis per provider


👩‍💻 Author
Kainat Fatima

GitHub: @kainatfatima
LinkedIn: linkedin.com/in/kainatfatima
