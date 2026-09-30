# Part 55: CI/CD กับ GitHub Actions

> **ระดับ:** มืออาชีพ / Professional  
> **Steps:** 1801-1840  
> **เวลาเรียน:** ~4 ชั่วโมง

---

## 📚 Table of Contents

1. [GitHub Actions คืออะไร](#github-actions-คืออะไร)
2. [Workflow YAML](#workflow-yaml)
3. [On Push, On PR Triggers](#triggers)
4. [Test + Lint + Build Pipeline](#pipeline)
5. [Deploy to Vercel via Actions](#deploy-vercel)
6. [Environment Secrets](#environment-secrets)
7. [Matrix Builds](#matrix-builds)
8. [Caching Dependencies](#caching-dependencies)
9. [Notifications](#notifications)
10. [Quiz](#quiz)

---

## Step 1801: GitHub Actions คืออะไร {#github-actions-คืออะไร}

GitHub Actions คือ CI/CD platform ที่ build ไว้ใน GitHub ช่วยให้ automate workflow ต่างๆ

```
CI/CD Pipeline ทั่วไป:
Developer Push Code
        │
        ▼
    GitHub Actions
        │
        ├──► Lint (ESLint, Prettier)
        ├──► Type Check (TypeScript)
        ├──► Unit Tests (Jest/Vitest)
        ├──► Integration Tests
        ├──► Build (Next.js)
        └──► Deploy (Vercel/AWS/GCP)
```

### ส่วนประกอบหลัก

```yaml
# โครงสร้าง Workflow
Workflow     # ไฟล์ YAML ใน .github/workflows/
└── Job      # งานที่ทำงานบน runner เดียว
    └── Step # แต่ละ action หรือ command
```

---

## Step 1802-1810: Workflow YAML {#workflow-yaml}

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline  # ชื่อ workflow

# Trigger
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  workflow_dispatch:  # Manual trigger

# Environment Variables ระดับ Workflow
env:
  NODE_VERSION: '18'
  PNPM_VERSION: '8'

# Jobs
jobs:
  # ============================================
  # Job 1: Setup (เตรียม environment)
  # ============================================
  setup:
    name: Setup
    runs-on: ubuntu-latest
    outputs:
      cache-key: ${{ steps.cache-key.outputs.value }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Get cache key
        id: cache-key
        run: echo "value=${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}" >> $GITHUB_OUTPUT
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Cache node_modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ steps.cache-key.outputs.value }}

  # ============================================
  # Job 2: Lint
  # ============================================
  lint:
    name: Lint
    runs-on: ubuntu-latest
    needs: setup
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Restore node_modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: Run ESLint
        run: pnpm lint
      
      - name: Check Formatting
        run: pnpm format:check

  # ============================================
  # Job 3: Type Check
  # ============================================
  type-check:
    name: Type Check
    runs-on: ubuntu-latest
    needs: setup
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Restore node_modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ needs.setup.outputs.cache-key }}
      - name: Run TypeScript
        run: pnpm type-check

  # ============================================
  # Job 4: Tests
  # ============================================
  test:
    name: Tests
    runs-on: ubuntu-latest
    needs: setup
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Restore node_modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ needs.setup.outputs.cache-key }}
      - name: Run Unit Tests
        run: pnpm test -- --coverage
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/coverage-final.json

  # ============================================
  # Job 5: Build
  # ============================================
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, type-check, test]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Restore node_modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ needs.setup.outputs.cache-key }}
      
      - name: Build
        run: pnpm build
        env:
          NEXT_PUBLIC_API_URL: ${{ secrets.NEXT_PUBLIC_API_URL }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-artifacts
          path: |
            .next/
            !.next/cache
          retention-days: 1
```

---

## Step 1811-1815: Triggers {#triggers}

```yaml
# ประเภท Triggers ต่างๆ
on:
  # Push ไปยัง branch
  push:
    branches:
      - main
      - develop
      - 'release/**'  # Pattern matching
    tags:
      - 'v*'          # Version tags
    paths:
      - 'src/**'      # เฉพาะเมื่อ src/ เปลี่ยน
      - '!**.md'      # ยกเว้น markdown files

  # Pull Request
  pull_request:
    types:
      - opened
      - synchronize  # เมื่อมี commit ใหม่
      - reopened
    branches:
      - main

  # Schedule (Cron)
  schedule:
    - cron: '0 0 * * *'  # ทุกวันตอนเที่ยงคืน UTC

  # Manual trigger
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      skip_tests:
        description: 'Skip tests'
        required: false
        type: boolean
        default: false

  # Trigger จาก workflow อื่น
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      DEPLOY_TOKEN:
        required: true

  # Trigger เมื่อ issue ถูกสร้าง
  issues:
    types: [opened, labeled]

  # Trigger เมื่อ PR ถูก merge
  pull_request_target:
    types: [closed]
```

---

## Step 1816-1820: Test + Lint + Build Pipeline {#pipeline}

### Full Pipeline

```yaml
# .github/workflows/full-pipeline.yml
name: Full CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Quality Checks
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      
      # Parallel steps ใช้ background processes
      - name: Run all quality checks
        run: |
          pnpm lint &
          LINT_PID=$!
          
          pnpm type-check &
          TYPECHECK_PID=$!
          
          wait $LINT_PID
          LINT_EXIT=$?
          
          wait $TYPECHECK_PID
          TYPECHECK_EXIT=$?
          
          exit $((LINT_EXIT + TYPECHECK_EXIT))
      
      - name: Run tests with coverage
        run: pnpm test:coverage
        env:
          DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
      
      - name: Build application
        run: pnpm build
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.NEXT_PUBLIC_API_URL }}
```

### E2E Testing กับ Playwright

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 6 * * *'  # ทุกวัน 6am

jobs:
  e2e:
    name: Playwright E2E
    runs-on: ubuntu-latest
    timeout-minutes: 60
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      
      - name: Install Playwright Browsers
        run: pnpm exec playwright install --with-deps chromium
      
      - name: Build app
        run: pnpm build
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.STAGING_API_URL }}
      
      - name: Run Playwright tests
        run: pnpm test:e2e
        env:
          BASE_URL: http://localhost:3000
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## Step 1821-1825: Deploy to Vercel {#deploy-vercel}

### Vercel Deployment

```yaml
# .github/workflows/deploy.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

jobs:
  deploy-preview:
    name: Deploy Preview
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - run: pnpm install -g vercel@latest
      
      - name: Pull Vercel Environment
        run: vercel pull --yes --environment=preview --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Build Project
        run: vercel build --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Deploy to Preview
        id: deploy
        run: |
          URL=$(vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }})
          echo "url=$URL" >> $GITHUB_OUTPUT
      
      - name: Comment PR with URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 Preview deployed: ${{ steps.deploy.outputs.url }}`
            })

  deploy-production:
    name: Deploy Production
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - run: pnpm install -g vercel@latest
      
      - name: Pull Vercel Environment
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Build Project
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Deploy to Production
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## Step 1826-1830: Environment Secrets {#environment-secrets}

### การจัดการ Secrets

```yaml
# วิธีใช้ Secrets ใน workflow
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # ใช้ environment-level secrets
    
    steps:
      - name: Example
        env:
          # Repository Secrets
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
          
          # Environment Secrets (environment-specific)
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
          
          # Organization Secrets
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
          
          # Variables (non-sensitive)
          API_URL: ${{ vars.NEXT_PUBLIC_API_URL }}
          APP_ENV: ${{ vars.APP_ENVIRONMENT }}
        run: |
          echo "Deploying to $APP_ENV..."
          # Never echo secrets directly!
```

### การ mask sensitive output

```yaml
- name: Get Token
  id: get-token
  run: |
    TOKEN=$(./scripts/get-token.sh)
    echo "::add-mask::$TOKEN"  # Mask ใน logs
    echo "token=$TOKEN" >> $GITHUB_OUTPUT

- name: Use Token
  run: |
    curl -H "Authorization: Bearer ${{ steps.get-token.outputs.token }}" \
      https://api.example.com/deploy
```

---

## Step 1831-1834: Matrix Builds {#matrix-builds}

Matrix Builds ช่วยทดสอบใน environments หลายๆ แบบพร้อมกัน

```yaml
# .github/workflows/matrix.yml
name: Matrix Build

on: [push]

jobs:
  test:
    name: Test on ${{ matrix.os }} / Node ${{ matrix.node }}
    runs-on: ${{ matrix.os }}
    
    strategy:
      fail-fast: false  # ไม่หยุดทุก jobs ถ้า job หนึ่งล้มเหลว
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: ['18', '20']
        # Exclude บาง combinations
        exclude:
          - os: windows-latest
            node: '18'
        # Include เพิ่มเติม
        include:
          - os: ubuntu-latest
            node: '20'
            experimental: true
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Use Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      
      - run: npm ci
      - run: npm test

  # Matrix สำหรับ Browser Testing
  e2e:
    name: E2E on ${{ matrix.browser }}
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        browser: [chromium, firefox, webkit]
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm exec playwright install ${{ matrix.browser }}
      - run: pnpm test:e2e --project=${{ matrix.browser }}
```

---

## Step 1835: Caching Dependencies {#caching-dependencies}

```yaml
# วิธี cache node_modules ต่างๆ

# 1. ใช้ actions/setup-node built-in cache
- uses: actions/setup-node@v4
  with:
    node-version: '18'
    cache: 'npm'      # หรือ 'yarn' หรือ 'pnpm'

# 2. Manual cache สำหรับ .next/cache
- name: Cache Next.js
  uses: actions/cache@v3
  with:
    path: |
      ~/.npm
      ${{ github.workspace }}/.next/cache
    key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
    restore-keys: |
      ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
      ${{ runner.os }}-nextjs-

# 3. Turborepo Remote Cache
- name: Run with Turborepo cache
  run: pnpm turbo build
  env:
    TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
    TURBO_TEAM: ${{ vars.TURBO_TEAM }}

# 4. pnpm store cache
- uses: pnpm/action-setup@v2
  with:
    version: 8
    run_install: false

- name: Get pnpm store directory
  id: pnpm-cache
  run: echo "dir=$(pnpm store path)" >> $GITHUB_OUTPUT

- uses: actions/cache@v3
  with:
    path: ${{ steps.pnpm-cache.outputs.dir }}
    key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
    restore-keys: ${{ runner.os }}-pnpm-store-

- run: pnpm install
```

---

## Step 1836-1840: Notifications {#notifications}

### Slack Notification

```yaml
# .github/workflows/notify.yml
- name: Notify Slack on Success
  if: success()
  uses: slackapi/slack-github-action@v1.24.0
  with:
    payload: |
      {
        "text": "✅ Deployment Successful!",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*✅ Deployment Successful!*\n*Branch:* ${{ github.ref_name }}\n*Commit:* ${{ github.sha }}\n*By:* ${{ github.actor }}"
            }
          },
          {
            "type": "actions",
            "elements": [
              {
                "type": "button",
                "text": { "type": "plain_text", "text": "View Deployment" },
                "url": "https://example.com"
              }
            ]
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK

- name: Notify Slack on Failure
  if: failure()
  uses: slackapi/slack-github-action@v1.24.0
  with:
    payload: |
      {
        "text": "❌ Deployment Failed!",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*❌ Deployment Failed!*\n*Branch:* ${{ github.ref_name }}\n*Commit:* ${{ github.sha }}\n*By:* ${{ github.actor }}\n*Logs:* <${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View Logs>"
            }
          }
        ]
      }
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

### Email Notification

```yaml
- name: Send Email on Failure
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 465
    username: ${{ secrets.MAIL_USERNAME }}
    password: ${{ secrets.MAIL_PASSWORD }}
    subject: "❌ Build Failed: ${{ github.repository }}"
    to: team@example.com
    from: noreply@example.com
    body: |
      Build failed for ${{ github.repository }}
      
      Branch: ${{ github.ref_name }}
      Commit: ${{ github.sha }}
      Author: ${{ github.actor }}
      
      View logs: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

---

## 🧪 Quiz - Part 55

**ข้อ 1:** GitHub Actions workflow file เก็บอยู่ที่ไหน?
- A) `.github/workflows/*.yml`
- B) `actions/*.yml`
- C) `ci/*.yml`
- D) `.circleci/config.yml`

**ข้อ 2:** Matrix builds ช่วยอะไร?
- A) เร็วขึ้น 2 เท่า
- B) ทดสอบใน configurations หลายๆ แบบพร้อมกัน
- C) Cache dependencies ดีขึ้น
- D) Deploy หลาย environments

**ข้อ 3:** วิธีใช้ secrets ใน workflow คือ?
- A) `${{ env.SECRET_NAME }}`
- B) `${{ secrets.SECRET_NAME }}`
- C) `process.env.SECRET_NAME`
- D) `$SECRET_NAME`

**ข้อ 4:** `needs: [lint, test]` ใน job หมายความว่า?
- A) Install lint และ test
- B) Job นี้จะรันหลังจาก lint และ test เสร็จ
- C) Job นี้ต้องการ Node.js
- D) Job นี้ Skip ถ้า lint หรือ test ล้มเหลว

**เฉลย:** 1-A, 2-B, 3-B, 4-B

---

> **➡️ Next:** [Part 56: Next.js Image Optimization](./part-56-nextjs-image-optimization.md)
