# Chapter 4: Azure Test Plans

## Overview
**Azure Test Plans** provides manual and exploratory testing tools within Azure DevOps. It enables test case management, test suites, test execution with traceability to work items, and integrates with Azure Pipelines for automated test reporting.

---

## 4.1 Test Plans Architecture

```
┌────────────── AZURE TEST PLANS ──────────────┐
│                                               │
│  Test Plan: "Sprint 5 Testing"                │
│  │                                            │
│  ├── Test Suite: Login Module                 │
│  │   ├── Test Case: TC01 - Valid login        │
│  │   ├── Test Case: TC02 - Invalid password   │
│  │   ├── Test Case: TC03 - Locked account     │
│  │   └── Test Case: TC04 - SSO login          │
│  │                                            │
│  ├── Test Suite: Payment Module               │
│  │   ├── Test Case: TC05 - Credit card pay    │
│  │   ├── Test Case: TC06 - PayPal pay         │
│  │   └── Test Case: TC07 - Refund process     │
│  │                                            │
│  └── Test Suite: Regression                   │
│      ├── (Shared steps across suites)         │
│      └── (Linked to requirements)             │
│                                               │
└───────────────────────────────────────────────┘
```

---

## 4.2 Types of Testing

| Test Type | Tool | Description |
|-----------|------|-------------|
| **Manual Testing** | Test Runner | Execute predefined test cases step by step |
| **Exploratory Testing** | Test & Feedback extension | Ad-hoc testing with screenshot/video capture |
| **Automated Testing** | Azure Pipelines | Unit, integration, UI tests run in pipelines |

```
┌────── TESTING WORKFLOW ───────────────────┐
│                                            │
│  1. Plan                                   │
│     ├── Create Test Plan                   │
│     ├── Define Test Suites                 │
│     └── Write Test Cases (steps + data)    │
│                                            │
│  2. Execute                                │
│     ├── Run manually (Test Runner)         │
│     ├── Explore (Test & Feedback ext.)     │
│     └── Automate (link to Pipeline tests)  │
│                                            │
│  3. Track                                  │
│     ├── Pass / Fail per test case          │
│     ├── File bugs for failures             │
│     └── Charts: pass rate, trend           │
│                                            │
└────────────────────────────────────────────┘
```

---

## 4.3 Writing Test Cases

```
┌──── TEST CASE: TC01 - Valid Login ──────────┐
│                                              │
│  Title: Verify valid user can login          │
│  Priority: 1 (Critical)                     │
│  Linked User Story: US#42                    │
│                                              │
│  Preconditions:                              │
│  • User account exists with valid creds      │
│  • Application is accessible                 │
│                                              │
│  Steps:                                      │
│  ┌────┬──────────────────┬────────────────┐  │
│  │ #  │ Action           │ Expected       │  │
│  ├────┼──────────────────┼────────────────┤  │
│  │ 1  │ Navigate to      │ Login page     │  │
│  │    │ /login           │ loads          │  │
│  ├────┼──────────────────┼────────────────┤  │
│  │ 2  │ Enter valid      │ Fields accept  │  │
│  │    │ username & pass   │ input          │  │
│  ├────┼──────────────────┼────────────────┤  │
│  │ 3  │ Click "Login"    │ Redirect to    │  │
│  │    │ button           │ dashboard      │  │
│  ├────┼──────────────────┼────────────────┤  │
│  │ 4  │ Verify welcome   │ "Welcome,      │  │
│  │    │ message          │ User" shown    │  │
│  └────┴──────────────────┴────────────────┘  │
│                                              │
│  Shared Steps: "Navigate to login" (reused)  │
│  Parameters: @username, @password            │
│                                              │
└──────────────────────────────────────────────┘
```

### Parameterised Test Cases

| Iteration | @username | @password | Expected Result |
|-----------|-----------|-----------|-----------------|
| 1 | user@test.com | Pass123! | Login success |
| 2 | admin@test.com | Admin456! | Login success |
| 3 | new@test.com | New789! | Login success |

---

## 4.4 Exploratory Testing

```
┌────── TEST & FEEDBACK EXTENSION ──────────┐
│                                            │
│  Browser Extension (Chrome / Edge)         │
│       │                                    │
│       ▼                                    │
│  Start Session                             │
│  ┌──────────────────────────────────────┐  │
│  │  • Browse application freely        │  │
│  │  • Capture screenshots              │  │
│  │  • Record screen video              │  │
│  │  • Add annotations/notes            │  │
│  │  • Create bugs directly from tool   │  │
│  │  • Create test cases from actions   │  │
│  │  • Timeline of actions captured     │  │
│  └──────────────────────────────────────┘  │
│       │                                    │
│       ▼                                    │
│  End Session → Summary of findings         │
│  • Bugs filed: 3                           │
│  • Test cases created: 2                   │
│  • Duration: 45 min                        │
│                                            │
└────────────────────────────────────────────┘
```

---

## 4.5 Test Automation Integration

```yaml
# Pipeline with automated test results
stages:
  - stage: Test
    jobs:
      - job: RunTests
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - script: |
              dotnet test --logger "trx;LogFileName=results.trx"
            displayName: 'Run automated tests'

          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '**/results.trx'
              mergeTestResults: true
              testRunTitle: 'Unit Tests - Sprint 5'
            condition: always()
```

```
┌────── TEST RESULT REPORTING ──────────────┐
│                                            │
│  Pipeline Run #200                         │
│  ┌──────────────────────────────────────┐  │
│  │  Tests Tab:                         │  │
│  │  ✅ 142 passed                      │  │
│  │  ❌   3 failed                      │  │
│  │  ⏭   1 skipped                     │  │
│  │  ──────────────────                 │  │
│  │  Pass Rate: 97.3%                   │  │
│  │                                     │  │
│  │  Failed Tests:                      │  │
│  │  • PaymentTest.RefundTimeout        │  │
│  │  • AuthTest.TokenExpiry             │  │
│  │  • UITest.DashboardLoad             │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

---

## 4.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Test Runner won't launch | Browser extension missing or disabled | Install Test & Feedback extension |
| Test results not in pipeline | Wrong test format or path | Verify `testResultsFormat` and file glob |
| Can't link test case to user story | Permissions or wrong link type | Use "Tested By/Tests" link type |
| Parameterised test not iterating | Data rows not configured | Add parameter rows in Test Case |
| Pass rate dropping | Flaky tests | Mark flaky, fix root cause, add retry |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Test Plans** | Organise tests into Plans → Suites → Cases |
| **Manual Testing** | Step-by-step execution with Test Runner |
| **Exploratory Testing** | Ad-hoc testing with Test & Feedback extension |
| **Test Cases** | Steps, expected results, parameters, shared steps |
| **Automation** | Pipeline tests published via `PublishTestResults` task |
| **Traceability** | Test Cases linked to User Stories and Bugs |

---

## Quick Revision Questions

1. **What is the hierarchy in Azure Test Plans?**
   > Test Plan → Test Suite → Test Case. Plans contain suites, suites contain cases with steps.

2. **What is exploratory testing?**
   > Unscripted testing using the Test & Feedback browser extension to discover bugs by freely exploring an application.

3. **How do you publish automated test results to Azure DevOps?**
   > Use the `PublishTestResults@2` task in a YAML pipeline, specifying the test format (VSTest, JUnit, NUnit) and file path.

4. **What are shared steps?**
   > Reusable step sequences (e.g., "Login as admin") that can be referenced by multiple test cases to avoid duplication.

5. **What are parameterised test cases?**
   > Test cases with data-driven iterations where parameters like @username and @password are substituted from a data table for each run.

---

[⬅ Previous: Azure Boards](03-azure-boards.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Artifacts ➡](05-azure-artifacts.md)
