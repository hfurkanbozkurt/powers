---
name: "aws-amplify"
displayName: "Build full-stack apps with AWS Amplify"
description: "Build full-stack web and mobile applications with AWS Amplify Gen 2 using type-safe TypeScript, guided workflows, and best practices for authentication, APIs, storage, and serverless functions"
keywords: ["amplify", "aws-amplify", "amplify gen 2", "gen2", "fullstack", "full-stack", "app", "application", "web", "mobile", "typescript", "react", "nextjs", "next.js", "vue", "nuxt", "angular", "react native", "flutter", "swift", "android", "ios"]
author: "AWS"
---

# AWS Amplify Gen 2

## Overview

Build full-stack applications with AWS Amplify Gen 2 using TypeScript code-first development. This power provides guided workflows for:

- Creating backend resources (auth, data, storage, functions)
- Deploying to sandbox and production environments
- Integrating frontend frameworks (React, Next.js, Vue, Angular, Flutter, Swift)
- Following Amplify Gen 2 best practices

## Getting Started

**For AI agents helping users build Amplify apps:**

First, validate these prerequisites with the user:

1. **Node.js 18.x or later**
   ```bash
   node --version
   ```

2. **AWS credentials configured**
   ```bash
   AWS_PAGER="" aws sts get-caller-identity
   ```
   If this fails, run `aws configure` or `aws sso login`.

3. **npm available**
   ```bash
   npm --version
   ```

Once prerequisites are confirmed, read the workflow steering file for step-by-step guidance:

```
Call action "readSteering" with powerName="aws-amplify", steeringFile="amplify-workflow.md"
```

The workflow will guide you through:
- Determining which phases apply to the user's request
- Presenting a plan and getting confirmation
- Executing phases: Backend → Sandbox → Frontend → Testing → Production
- Calling the appropriate SOPs for each phase

## Available Steering Files

This power has one steering file:

- **amplify-workflow** - Orchestrated workflow for Amplify Gen 2 development. Coordinates phases: Backend → Sandbox → Frontend → Testing → Production.
