
| Field | Value |
|---|---|
| Document ID | MTP-CPP-001 |
| Version | 1.0 |
| Author | Maryna (QA Engineer) |
| Status | Approved |
| Application under test | Clinical Placement Platform (web application) |
| Document type | Master Test Plan |




## Contents

1. [Introduction](#1-introduction)
2. [Test Items](#2-test-items)
3. [Scope](#3-scope)
4. [Test Levels](#4-test-levels)
5. [Test Types](#5-test-types)
6. [Test Design and Prioritization](#6-test-design-and-prioritization)
7. [Automation Approach](#7-automation-approach)
8. [Test Environments](#8-test-environments)
9. [Test Data Management](#9-test-data-management)
10. [Entry and Exit Criteria](#10-entry-and-exit-criteria)
11. [Suspension and Resumption Criteria](#11-suspension-and-resumption-criteria)
12. [Defect Management](#12-defect-management)
13. [Roles and Responsibilities](#13-roles-and-responsibilities)
14. [Test Process and Schedule](#14-test-process-and-schedule)
15. [Metrics and Reporting](#15-metrics-and-reporting)
16. [Communication Plan](#16-communication-plan)
17. [Test Deliverables](#17-test-deliverables)
18. [Risks and Mitigations](#18-risks-and-mitigations)
19. [Assumptions and Dependencies](#19-assumptions-and-dependencies)
20. [Approvals](#20-approvals)


## 1. Introduction

### 1.1 Purpose
This Master Test Plan (MTP) defines the overall testing approach for the Clinical Placement Platform across its whole lifecycle. It describes all test levels, test types, environments, roles, processes and quality criteria that apply to every release.

Detailed, release-specific planning is done in level test plans (see 1.4). They follow the rules set in this document.

### 1.2 Product overview
The Clinical Placement Platform connects nursing students (Nurse Practitioner, APRN and RN programs) with clinical preceptors who supervise their required clinical hours.

| Role | What the user does |
|---|---|
| Student | Registers, completes a profile, uploads required documents, searches for preceptors, requests and pays for a clinical rotation, logs clinical hours |
| Preceptor | Registers, sets specialty and availability, accepts or declines rotation requests, approves logged hours, completes student evaluations |
| Admin (CC) | Reviews documents, manages matches and rotations, handles exceptions and support requests |

### 1.3 Objectives
- Define one consistent testing approach for the whole product.
- Make sure every feature is tested at the right level, by the right people.
- Detect defects as early as possible (shift-left: QA takes part in requirements and design reviews).
- Make sure each role can see and do only what that role is allowed to.
- Build and maintain an automated regression suite for critical flows.
- Give stakeholders clear, regular information about product quality and release readiness.

### 1.4 Related documents
| Document | Purpose | Location |
|---|---|---|
| Release test plan | Scope, schedule and resources for a specific release | `test-plan/` |
| Regression test plan | Regression suite content and execution rules | `test-plan/` |
| UAT plan | Business scenarios and sign-off process | `test-plan/` |
| Manual test cases | Detailed test cases per module | `zephyr/test-cases/manual_test_cases/` |
| API test cases | Endpoint checks | `zephyr/test-cases/api_test_cases/` |
| SQL data checks | Data integrity queries | `zephyr/test-cases/sql_test_cases/` |
| Automation project | Automated UI and API tests | `playright/test-cases/e2e_test_cases/` |


## 2. Test Items

| ID | Module | Description |
|---|---|---|
| M-01 | Registration and login | Sign up for students and preceptors, email verification, login, password reset, logout |
| M-02 | Student profile | Personal info, school and program, required clinical hours, specialty needed, preferred location and dates |
| M-03 | Preceptor profile | Credentials, specialty, practice location, availability calendar, capacity (number of students) |
| M-04 | Document management | Upload of required documents (e.g., immunization record, CPR certificate, school affiliation letter), file type and size validation, document status (Pending / Approved / Rejected / Expired) |
| M-05 | Preceptor search | Search and filters (specialty, location, distance, dates, availability), sorting, results visibility rules |
| M-06 | Rotation request and matching | Student sends request, preceptor accepts/declines, request expiration, admin manual matching |
| M-07 | Payments | Pricing display, payment checkout, payment confirmation, refunds/cancellation rules |
| M-08 | Rotation management | Rotation status (Requested → Confirmed → In progress → Completed / Cancelled), start/end dates, rescheduling |
| M-09 | Hours logging and evaluations | Student logs clinical hours, preceptor approves/rejects, final evaluation, hours summary |
| M-10 | Notifications | Email and in-app notifications for key events (request sent, accepted, document rejected, payment received, hours approved) |
| M-11 | Admin panel | User management, document review, match management, rotation overrides, reports |


## 3. Scope

### 3.1 In scope
- All modules in section 2, for all three roles
- All test levels described in section 4
- Functional, UI, API, database, notification and role-based access testing
- Feature-flag-controlled functionality in both ON and OFF states
- Cross-browser and responsive testing
- Regression testing for every release and hotfix
- Test automation for critical and repeated scenarios
- Support of User Acceptance Testing

### 3.2 Out of scope
| Item | Reason |
|---|---|
| Penetration and security testing | Performed by a separate security team |
| Load and performance testing | Planned as a separate activity with its own plan |
| Third-party payment provider internals | Tested only on our side (sandbox mode) |
| Third-party email delivery | We verify that the email is triggered and its content, not the provider's delivery |


## 4. Test Levels

| Level | Goal | Who | Environment | Entry | Exit |
|---|---|---|---|---|---|
| **Unit testing** | Check functions and components in isolation | Developers | Local / CI | Code written | Unit tests pass in CI, agreed coverage reached |
| **Integration testing** | Check communication between UI, API, database and third-party services | Developers + QA | Development / Staging | Unit tests pass, build deployed | Main integrations work, no Critical defects |
| **System testing** | Check each module and the full system against requirements | QA | Staging | Smoke test passed, test cases ready | Exit criteria in section 10 met |
| **End-to-end testing** | Check complete user journeys across roles | QA | Staging | System testing of related modules done | All P1 E2E scenarios passed |
| **Regression testing** | Confirm that new changes did not break existing features | QA (mostly automated) | Staging | Release candidate build ready | Regression suite passed or failures accepted |
| **User Acceptance Testing (UAT)** | Confirm the product meets business needs | Product Owner, UAT stakeholders, QA support | Staging / UAT | System testing complete, no open Critical defects | Sign-off received |
| **Post-release verification** | Quick check that the release works in production | QA | Production | Release deployed | Smoke checks passed |


## 5. Test Types

| Type | What is checked | Level | Manual / Automated |
|---|---|---|---|
| Smoke | Login for each role, search, create a request | System, Production | Automated |
| Functional | Business rules of each module | System | Manual, critical cases automated |
| Negative | Invalid/missing data, wrong file types, expired documents, requests to unavailable preceptors | System | Manual + automated |
| Role-based access | Pages, buttons and data available per role; direct URL access by the wrong role | System | Manual + API |
| API | Status codes, response body, validation errors, auth required | Integration, System | Postman + pytest |
| Database | Records created/updated correctly after UI actions | Integration, System | SQL queries |
| Notifications | Correct trigger, recipient and content of emails | System | Manual |
| Feature flags | Behavior with flag ON and OFF | System | Manual |
| UI / Usability | Layout, labels, error messages, consistency with design | System | Manual |
| Cross-browser / Responsive | Main flows on supported browsers and screen sizes | System | Automated + manual |
| Exploratory | Time-boxed sessions per module or new feature | System | Manual |
| Regression | Existing functionality after changes | Regression | Automated + manual |
| Acceptance | Business scenarios | UAT | Manual |


## 6. Test Design and Prioritization

### 6.1 Test design techniques
- **Equivalence partitioning** – valid / invalid / empty values in registration and profile forms.
- **Boundary value analysis** – required hours (0, 1, max), preceptor capacity (0, 1, full), file size limit (just below, at, just above).
- **Decision table** – whether a preceptor appears in search results, based on profile status, availability, specialty and capacity.
- **State transition** – rotation status, document status, request status (Pending → Accepted / Declined / Expired).
- **Use case testing** – end-to-end journeys for each role.
- **Error guessing** – double-clicking Pay, using browser Back after payment, uploading a renamed file, time zone differences in dates.
-  **Exploratory testing** - defining a new scenarios.

### 6.2 Risk-based prioritization
| Priority | Meaning | Examples |
|---|---|---|
| P1 – Critical | Blocks the placement flow or affects payments | Login, rotation request, payment, preceptor accept |
| P2 – High | Main business rule works incorrectly | Search filters, availability, hours totals, document expiration |
| P3 – Medium | Secondary feature | Profile editing, sorting, notification wording |
| P4 – Low | Cosmetic | Labels, alignment, tooltips |

### 6.3 Test case management
- Test cases are stored in Zephyr Scale and linked to user stories in Jira.
- Each test case has an ID, priority, preconditions, steps, test data and expected result.
- Test cases are reviewed before execution and updated when requirements change.
- A requirements traceability matrix links user stories → test cases → defects.


## 7. Automation Approach

| Item | Approach |
|---|---|
| UI automation | Python, Playwright, pytest, Page Object Model |
| API automation | Postman collections; pytest + requests |
| What to automate | Smoke suite, P1/P2 regression cases, stable and repeated scenarios, cross-browser runs |
| What not to automate | Exploratory testing, email content review, UI look-and-feel, one-time checks |
| Execution | On every deployment to staging (smoke) and before every release (full regression) |
| Reporting | pytest-html report; screenshots and traces on failure |
| Maintenance | Failing tests are reviewed within one day: product defect → bug in Jira; test issue → fixed in the test |


## 8. Test Environments

| Environment | Purpose | Data | Who uses it |
|---|---|---|---|
| Local | Development and unit tests | Mock data | Developers |
| Development | Integration of new features | Synthetic data | Developers, QA (early checks) |
| Staging | System, E2E, regression testing and UAT | Synthetic, production-like data | QA, Product Owner, UAT stakeholders |
| Production | Post-release smoke checks | Real data (read-only checks with test accounts) | QA |

**Supported platforms**

| Item | Details |
|---|---|
| OS | Windows 11 |
| Browsers | Chrome, Firefox, Edge) |
| Viewports | 1920×1080, 1366×768, 768×1024 (tablet), 390×844 (mobile) |
| Payments | Payment provider sandbox mode with test cards |
| Emails | Test mailbox / email capture tool |

**Tools:** Jira, Confluence, Zephyr Scale, Postman, SQL client, Chrome DevTools, VS Code, Playwright, Claude code.


## 9. Test Data Management

- Only synthetic test data is used. No real student, preceptor or patient information is used in test environments.
- Test accounts use a dedicated test email domain.
- Uploaded documents are dummy files.
- Test data is created through the UI, API or SQL scripts and cleaned up after each test cycle.
- Test data sets are versioned together with the test cases.

**Standard test accounts**

| Account | Role | State | Used for |
|---|---|---|---|
| student_new | Student | Just registered, empty profile | Onboarding, required fields |
| student_ready | Student | Profile complete, all documents approved | Search, request, payment |
| student_expired_docs | Student | One document expired | Blocking rules, notifications |
| preceptor_available | Preceptor | Approved, has availability and capacity | Search results, accept flow |
| preceptor_full | Preceptor | Capacity reached | Not shown in search / cannot accept |
| preceptor_pending | Preceptor | Profile not yet approved | Must not appear in search |
| admin_user | Admin | Full access | Document review, manual matching |


## 10. Entry and Exit Criteria

These criteria apply to system testing for every release. Criteria for other levels are in section 4.

### 10.1 Entry criteria
- Release scope is defined, and user stories have acceptance criteria.
- Build is deployed to staging, and the smoke test passed.
- Test cases are written and reviewed.
- Test accounts and test data are ready.
- Feature flag states for the release are known.

### 10.2 Exit criteria
- 100% of planned P1 and P2 test cases executed.
- At least 95% of executed test cases passed.
- No open Critical or High defects (or accepted by the Product Owner with a workaround).
- Automated regression suite passed.
- UAT completed and signed off.
- Test summary report shared with the team.


## 11. Suspension and Resumption Criteria

**Testing is suspended if:**
- The test environment is unavailable.
- The smoke test fails (e.g., login or search does not work).
- A blocker defect stops more than 30% of planned test cases.
- A wrong build is deployed to the test environment.

**Testing resumes when:**
- The environment is restored, or the blocker is fixed and verified.
- The smoke test passes again.


## 12. Defect Management

### 12.1 Defect workflow
`New → Open → In Progress → Ready for QA → Verified → Closed`
A defect can also be moved to `Reopened`, `Rejected` or `Deffered`.

### 12.2 Severity levels
| Severity | Description | Example |
|---|---|---|
| Critical | Main flow is blocked or users see data they should not see; no workaround | Student cannot pay; a user can open another user's profile |
| High | Main feature fails; workaround is difficult | Unavailable preceptor appears in search results |
| Medium | Feature partly works; workaround exists | Sorting by distance ignores one filter |
| Low | Cosmetic issue | Typo in an email template |

### 12.3 Defect report fields
Summary, Environment, Role/account used, Preconditions, Steps to reproduce, Actual result, Expected result, Severity, Attachments (screenshot, video, logs), Environment, Related user story / test case.

### 12.4 Defect triage
Open defects are reviewed with the Project Manager regularly to agree on priority and the target release.


## 13. Roles and Responsibilities

| Role | Responsibilities |
|---|---|
| QA Engineer | Owns this Master Test Plan and level test plans; reviews requirements; designs and runs tests; maintains automation; reports defects and quality status |
| Tech Lead | Reviews the test approach, supports technical investigation, approves release readiness |
| Developers | Unit and integration tests, defect fixes, deployments to test environments |
| Designer | Clarifies UI/UX expectations, reviews UI defects |
| Product Owner | Defines acceptance criteria, prioritizes defects, signs off on UAT |
| UAT stakeholders | Run business scenarios and confirm the release is ready |


## 14. Test Process and Schedule

Testing follows the team's Agile process. Each sprint and release goes through the same steps.

| Phase | Activities | When | Output |
|---|---|---|---|
| Requirements review | QA reviews user stories and designs, asks questions, checks acceptance criteria | Backlog refinement, before sprint start | Clarified stories, testability notes |
| Test planning | Release test plan based on this MTP | Sprint start | Release test plan |
| Test design | Test cases, test data, automation updates | During development | Reviewed test cases |
| Test execution | Feature testing as stories are delivered to staging | During sprint | Results, defects |
| Regression | Automated suite + manual P1 checks | Before release | Regression results |
| UAT | Business validation with stakeholders | Before release | UAT sign-off |
| Release | Go/no-go decision, deployment, post-release smoke | Release day | Production smoke results |
| Retrospective | Review of escaped defects and process improvements | After release | Action items |

**Hotfixes:** focused testing of the fix + smoke suite + affected-area regression.


## 15. Metrics and Reporting

| Metric | Why it is tracked |
|---|---|
| Test execution progress (planned / executed / passed / failed / blocked) | Shows testing status at a glance |
| Requirements coverage | Every user story has at least one test case |
| Open defects by severity | Release readiness |
| Defect leakage (defects found in production) | Quality of testing and process |
| Defect reopen rate | Quality of fixes |
| Automated regression pass rate and run time | Health of the automation suite |

| Report | Audience | Frequency |
|---|---|---|
| Daily QA status | Team | Daily stand-up |
| Test summary report with go/no-go recommendation | Product Owner, Tech Lead | Each release |
| Quality trends (leakage, defects per release) | Team, management | Monthly |


## 16. Communication Plan

| What | Channel | Who |
|---|---|---|
| Daily testing status and blockers | Stand-up, team chat | QA → Team |
| Defects | Jira | QA ↔ Developers |
| Requirement questions | Jira comments, refinement meetings | QA ↔ Product Owner, Designer |
| Release readiness | Go/no-go meeting, test summary report | QA → Product Owner, Tech Lead |
| UAT coordination | Meeting, shared checklist | QA ↔ UAT stakeholders |


## 17. Test Deliverables

| Deliverable | When |
|---|---|
| Master Test Plan | Project start; updated when the process changes |
| Release / level test plans | Each release |
| Test cases (manual, E2E, API) and traceability matrix | Before execution; maintained continuously |
| Test data and SQL scripts | Before execution |
| Defect reports | During testing |
| Automated test suite and reports | Continuously |
| Daily status updates | During testing |
| Test summary report with go/no-go recommendation | Each release |


## 18. Risks and Mitigations

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Unclear or changing requirements | High | Medium | QA joins refinement; open questions confirmed with the Product Owner in writing |
| Several user roles increase test combinations | High | High | Role/permission matrix; prioritize critical paths per role |
| Staging data differs from production | Medium | Medium | Realistic synthetic data sets; SQL checks before testing |
| Third-party services (payments, email) unavailable in staging | Medium | Low | Sandbox modes; mocks where possible; retest when the service is back |
| Feature flags create hidden combinations | Medium | Medium | Test each flagged feature in ON and OFF states; record flag state in every test run |
| Date and time zone issues (availability, rotation dates) | Medium | Medium | Dedicated test cases for time zones and date boundaries |
| Short testing window before release | Medium | Medium | Risk-based prioritization; automated regression |
| Automation becomes flaky or outdated | Medium | Medium | Page Object Model; review failures within one day; regular maintenance |


## 19. Assumptions and Dependencies

- User stories include acceptance criteria before development starts.
- Test environments are stable and have production-like configuration.
- Payment and email providers have working sandbox/test modes.
- Developers are available to fix Critical and High defects during the test cycle.
- UAT stakeholders are available within the planned UAT window.


## 20. Approvals

| Name | Role | Date | Signature |
|---|---|---|---|
| Maryna | QA Engineer | | |
| | Tech Lead | | |
| | Product Owner | | |

