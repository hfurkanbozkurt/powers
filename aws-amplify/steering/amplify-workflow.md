# Amplify Workflow

Orchestrated workflow for AWS Amplify Gen 2 development.

## When to Use This Workflow

Use for any Amplify Gen 2 work:
- Building a new full-stack application
- Adding features to an existing backend
- Connecting frontend to backend
- Deploying to sandbox or production

The workflow determines which phases apply based on your request.

---

## Critical Rules

1. **Always follow SOPs completely** - Do not improvise or skip steps
2. **Never use Gen 1 patterns** - This power is for Amplify Gen 2 only (TypeScript code-first, `defineAuth`/`defineData`/`defineStorage`/`defineFunction`)
3. **If you encounter an error or get sidetracked:**
   - Fix the immediate issue
   - Return to the SOP and continue from where you left off
   - Do NOT abandon the SOP or start improvising
4. **If you lose track of where you were in the SOP:**
   - Call `retrieve_agent_sop` again to get the full SOP
   - Identify which step you completed last
   - Continue from the next step

---

## Step 1: Determine Applicable Phases

Based on the user's request and project state, determine which phases apply:

| Phase | Applies when |
|-------|--------------|
| 1: Backend | User needs to create or modify Amplify backend resources |
| 2: Sandbox | Backend code needs deployment for testing |
| 3: Frontend | Frontend needs to connect to Amplify backend |
| 4: Testing | App ready for local verification |
| 5: Production | User wants to deploy to production |

Common patterns:
- **New full-stack app:** 1 → 2 → 3 → 4 → 5
- **Add feature to existing backend:** 1 → 2
- **Redeploy after changes:** 2 only
- **Connect existing frontend:** 3 → 4
- **Deploy to production:** 5 only

---

## Step 2: Present Plan and Confirm

Present to the user:

```
📋 Here's my plan:

**What I understood:**
- [Brief summary of what the user wants]

**Features:** [list features if applicable]

**Framework:** [framework if known]

**Phases I'll execute:**
1. [Phase name] - [one-line description] → SOP: [sop-name]
2. [Phase name] - [one-line description] → SOP: [sop-name]
...
(Include SOP name for phases 1, 2, 3, and 5. Phase 4 has no SOP.)

Sound good?
```

**WAIT for user confirmation before proceeding.**

---

## Step 3: Execute Phases

Execute each applicable phase IN SEQUENCE.

**When starting a phase, announce it:**
```
⚙️ Phase 1: Backend (SOP: amplify-backend-implementation) - [Next: Phase 2: Sandbox Deployment]
🚀 Phase 2: Sandbox Deployment (SOP: amplify-deployment-guide) - [Next: Phase 3: Frontend Integration]
🎨 Phase 3: Frontend Integration (SOP: amplify-frontend-integration) - [Next: Phase 4: Local Testing]
🧪 Phase 4: Local Testing - [Next: Phase 5: Production Deployment]
🌐 Phase 5: Production Deployment (SOP: amplify-deployment-guide)
```
Omit "[Next: ...]" if it's the last phase in your plan.

---

### Phase 1: Backend

⚠️ **Do NOT write any code until you have called `retrieve_agent_sop` and read the response.**

```
Call `retrieve_agent_sop` with sop_name: "amplify-backend-implementation"
Follow the SOP completely.
```

**After completion:**
- Summarize what was created
- If Phase 2 is next: "Ready to deploy to sandbox? 🚀"

---

### Phase 2: Sandbox Deployment

⚠️ **Do NOT run any commands until you have called `retrieve_agent_sop` and read the response.**

```
Call `retrieve_agent_sop` with sop_name: "amplify-deployment-guide"
Follow the SOP for SANDBOX deployment.
```

**After completion:**
- Confirm deployment succeeded and `amplify_outputs.json` exists
- If Phase 3 is next: "Ready to connect the frontend? 🎨"

---

### Phase 3: Frontend Integration

**Prerequisite:** `amplify_outputs.json` must exist. If not, run Phase 2 first.

⚠️ **Do NOT write any code until you have called `retrieve_agent_sop` and read the response.**

```
Call `retrieve_agent_sop` with sop_name: "amplify-frontend-integration"
Follow the SOP completely.
```

**After completion:**
- Summarize integration work
- If Phase 4 is next: "Time to test! 🧪"

---

### Phase 4: Local Testing

Present to the user:

```
🧪 Time to test!

Start your dev server:
[framework-specific command]

Then try out:
[list features implemented]

Let me know how it goes!
```

**WAIT for user confirmation before proceeding to Phase 5.**

---

### Phase 5: Production Deployment

⚠️ **Do NOT run any commands until you have called `retrieve_agent_sop` and read the response.**

```
Call `retrieve_agent_sop` with sop_name: "amplify-deployment-guide"
Follow the SOP for PRODUCTION deployment.
```

**After completion:**

```
🎉 You're live!

**Production URL:** [url]
**Amplify Console:** https://console.aws.amazon.com/amplify/home
```

---

## Troubleshooting

If issues occur during any phase:
1. Check the SOP's troubleshooting section first
2. Search docs: `search_documentation(search_phrase: "error message", topics: ["amplify_docs"])`
3. Read relevant page with `read_documentation`

**After resolving the issue, immediately return to the SOP and continue from where you left off. Do not abandon the workflow.**
