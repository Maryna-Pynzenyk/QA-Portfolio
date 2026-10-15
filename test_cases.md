# Test cases: Clinical Placement Platform

| Field | Value |
|---|---|
| Application | Clinical Placement Platform (web application + API) |
| Author | Maryna (QA Engineer) |


## Contents

| Section | Count | Link |
|---|---|---|
| Test cases | 7 | [open](#test-cases) |
| BDD test cases | 9 features | [open](#bdd-test-cases) |

## Test cases

| ID | Title |
|---|---|
| [TC-RES-001](#tc-res-001) | Verify that a preceptor who cannot take more students in this date range is not available for students to reserve |
| [TC-RES-002](#tc-res-002) | Verify that a student is not able to see the preceptor's profile until the last payment is charged |
| [TC-API-001](#tc-api-001) | Verify that an admin is not able to save rotation hours via API with zero, negative or non-numeric values |
| [TC-API-002](#tc-api-002) | Verify that a CC is able to get the students list via API filtered by multiple Programs of Study with correct pagination |
| [TC-DB-001](#tc-db-001) | Verify that a new student record is saved with the correct data after sign-up |
| [TC-BE-001](#tc-be-001) | Verify that the system updates the payment and rotation status when a payment event is received |
| [TC-SEC-001](#tc-sec-001) | Verify that a student is not able to see another student's data by changing the record ID |

### TC-RES-001

**Title:** Verify that a preceptor who cannot take more students in this date range is not available for students to reserve
| | |
|---|---|
| Priority | P1 |
| Preconditions | Student is signed up. The student's wizard is completed. The Home page is opened. Preceptor is public and available; max number of students = 1. |
| Test data |Test card for a successful payment |

| # | Steps| Expected result |
|---|---|---|
| 1 | Search the preeceptor by Id. | The preceptor is found. |
| 2 | Click the Reserve button. | The reservation and payment page is opened. |
| 3 | Select the rotation dates and hours (Mar 2 – Apr 24, 120 hours).| The dates and horse are setlected. |
| 4 | Add the payment card. | The card is added. |
| 5 | Click the [Reserve] button. | The payment is charged. The rotation has Deposit Paid status. |
| 6 | On the Home page, search for the same preceptor again by ID. | P1 is found by ID. |
| 7 | Click the [Reserve] button. | The reservation and payment page is opened. The payment card is already predefined to the default value that was charged previously. |
| 8 | Select the same dates and hours as in step 3 (Mar 2 – Apr 24, 120 hours). | Message that the preceptor has no available spots for these dates. |
| 9 | Try to click [Reserve] to finish the reservation. | Payment is not taken, and the reservation is not created. |

### TC-RES-002

**Title:** Verify that a student is not able to see the preceptor's profile until the last payment is charged
|---|---|
| Priority | P1 |
| Preconditions | Student is signed up. The wizard is completed. The rotation is created. The system has a public and available preceptor. The student is logged in to the app. |
| Test data | Test card for a successful payment |

| # | Steps | Expected result |
|---|---|---|
| 1 | Open a created rotation. | The rotation is opened. |
| 2 | Select the available preceptor from the list. | The preceptor is selected. The preceptor's profile is opened. |
| 3 | Click the [Create a reservation] button. | The reservation and payment page is opened. |
| 4 | Select the rotation dates and hours. | The dates and horse are selected. |
| 5 | Add the payment card. | The card is added. |
| 6 | Click the [Reserve] button. | The payment is charged. The rotation has Deposit Paid status. No preceptor's info is shown to the student.|
| 7 | Navigate to the Admin panel and log in as CC. | The Rotations tab page is opened, and CC is logged in. |
| 8 | Search for the created and paid rotation by ID. |  The rotation is shown in the list. | 
| 9 | Open the rotation. | The rotation  is opened. |
| 10 | Move the rotation forward using the timeline to charge the next payment. | The rotation status is changed to The Second Payment Paid. The payment is charged. |
| 11 | Navigate back to the rotation view in the student's profile. | The rotation page is opened. The new status is shown. Preceptor's name, contacts and profile link are now shown. 

### TC-API-001

**Title:** Verify that an admin is not able to save rotation hours via API with a negative value
|---|---|
| Priority | P2 |
| Tool | Postman |
| Request | `POST {{baseUrl}}/rotations/{{rotationId}}/hours` |
| Preconditions | Postman environment with `baseUrl` is selected. Valid `adminToken` is saved in the environment. A new rotation exists, its ID is saved to `rotationId`. |
| Test data | `{"hours": -40}` |

| # |Steps| Expected result |
|---|---|---|
| 1 | Create a new `POST` request with the URL `{{baseUrl}}/rotations/{{rotationId}}/hours`. | URL is resolved with the values from the environment. |
| 2 | Open the Authorization tab, select Bearer Token and enter `{{adminToken}}`. | Token is added to the request headers. |
| 3 | Open the Body tab, select raw → JSON and enter `{"hours": -40}`. | Body is valid JSON. |
| 4 | Click Send. | Status is `400 Bad Request`. |
| 5 | Check the response body. | Response contains the error "Hours must be a positive number." |
| 6 | Send `GET {{baseUrl}}/rotations/{{rotationId}}` with the same token. | Status is `200 OK`. `hours` is not `-40`, the negative value was not saved. |

### TC-API-002

**Title:** Verify that a CC is able to get the students list via API filtered by two Programs of Study
|---|---|
| Priority | P2 |
| Tool | Postman |
| Request | `GET {{baseUrl}}/students?programOfStudy=3&programOfStudy=10` |
| Preconditions | Postman environment with `baseUrl` is selected. Valid `adminToken` (CC) is saved in the environment. Test students exist in programs with IDs 3 and 10, and in at least one other program (for example 12). |
| Test data | `programOfStudy` = 3 and 10 |

| # | Steps | Expected result |
|---|---|---|
| 1 | Create a new `GET` request with the URL `{{baseUrl}}/students`. | URL is resolved with the value from the environment. |
| 2 | Open the Authorization tab, select Bearer Token and enter `{{adminToken}}`. | Token is added to the request headers. |
| 3 | Open the Params tab and add two keys: `programOfStudy` = `3` and `programOfStudy` = `10`. | URL changes to `{{baseUrl}}/students?programOfStudy=3&programOfStudy=10`. |
| 4 | Click Send. | Status is `200 OK`. |
| 5 | Check the response body. | No `errors` field in the response. The list is not empty. |
| 6 | Check `programOfStudy` of every student in the list. | Every student has `programOfStudy` 3 or 10. No students from other programs (for example 12). |
| 7 | Compare the number of students in the response with the number of test students in programs 3 and 10. | The numbers match. |

### TC-DB-001

**Title:** Verify that a new student record is saved in the database with the correct data after sign-up
|---|---|
| Priority | P1 |
| Tool | SQL client
| Preconditions | Email `student.qa+db001@test.example` is not registered. Program of Study "Family Nurse Practitioner" exists. |
| Test data | Email: `student.qa+db001@test.example`, First name: Anna, Last name: Smith, Program of Study: Family Nurse Practitioner |

| # | Steps | Expected result |
|---|---|---|
| 1 | In the app, sign up as a student with the test data. | Sign-up is successful. |
| 2 | In the SQL client, run:<br>`SELECT id, email, role FROM users WHERE LOWER(email) = LOWER('student.qa+db001@test.example');` | Exactly 1 row is returned. `email` = `student.qa+db001@test.example`, `role` = STUDENT. |
| 3 | Copy the `id` from step 2 and run:<br>`SELECT s.first_name, s.last_name, p.name, s.status FROM students s JOIN programs_of_study p ON p.id = s.program_of_study_id WHERE s.user_id = <id from step 2>;` | Exactly 1 row is returned: Anna, Smith, Family Nurse Practitioner, ACTIVE. |

### TC-BE-001

**Title:** Verify that the system updates the payment and rotation status when a successful payment event is received
|---|---|
| Priority | P1 |
| Tools | App, payment provider dashboard (test mode), SQL client, Postman |
| Preconditions | Student is signed up. The wizard is completed. The rotation is created. The system has a public and available preceptor. The preceptor is added to the rotation, and the rotation is waiting for payment. |
| Test data | Test card for a successful payment |

| # | Steps| Expected result |
|---|---|---|
| 1 | Open a created rotation. | The rotation is opened. |
| 2 | Select the available preceptor from the list. | The preceptor is selected. The preceptor's profile is opened. |
| 3 | Click the [Create a reservation] button. | The reservation and payment page is opened. |
| 4 | Select the rotation dates and hours. | The dates and horse are selected. |
| 5 | Add the payment card. | The card is added. |
| 6 | Click the [Reserve] button. | The payment is charged. The rotation has Deposit Paid status. |
| 7 | Open Stripe and look for the payment Id to see the status. | The payment status is Successful. |
| 8 | In the database, run:<br>`SELECT rotation_id, amount, status FROM payments WHERE rotation_id = 'R1';` | Exactly 1 row is returned. `status` = PAID, `amount` matches the rotation price. |
| 9 | In Postman, send `GET {{baseUrl}}/rotations/R1` with the student's token. | Status is `200 OK`. Rotation `status` = CONFIRMED. |
| 10 | In the app, refresh the rotation page. | Rotation is shown as Deposit Paid. |

### TC-SEC-001

**Title:** Verify that a student is not able to see another student's data by changing the record ID
|---|---|
| Priority | P1 |
| Tool | Postman |
| Preconditions | Student 1 is signed up. Student B is signed up. Both students have the wizard completed. Both students have the rotations created. Logged in as Student 1. |

| # | Steps | Expected result |
|---|---|---|
| 1 | Create a new `POST` request with the URL `{{baseUrl}}/auth/login`. | URL is resolved with the value from the environment. |
| 2 | Open the Body tab, select raw → JSON and enter:<br>`{"email": "<student 1 email>", "password": "<student 1 password>"}` | Body is valid JSON. |
| 3 | Click Send. | Status is `200 OK`. Response contains `accessToken`. |
| 4 | Copy `accessToken` from the response and save it in the environment as `token_1`. | `token_1` is saved. |
| 5 | Create a new `GET` request with the URL `{{baseUrl}}/rotations/{{rotationId_1}}`. | URL is resolved with Student 1's rotation ID. |
| 6 | Open the Authorization tab, select Bearer Token and enter `{{token_1}}`. | Token is added to the request headers. |
| 7 | Click Send. | Status is `200 OK`. Response contains Student 1's rotation data. |
| 8 | In the same request, change the URL to `{{baseUrl}}/rotations/{{rotationId_2}}`. Keep `{{token_1}}` in the Authorization tab. | URL is resolved with Student 2's rotation ID. |
| 9 | Click Send. | Status is `403 Forbidden` or `404 Not Found`. |
| 10 | Check the response body. | No data of Student 2: no name, dates, preceptor, documents or payment details. Only an error message. |
| 11 | Change the URL back to `{{baseUrl}}/rotations/{{rotationId_1}}` and click Send. | Status is `200 OK`. Student 1's rotation data is returned, so the token still works. |


## BDD test cases

| Feature |
|---|
| [Student registration and login](#feature-student-registration-and-login) |
| [Rotation request flow](#feature-rotation-request-flow) |
| [Last-minute fee](#feature-last-minute-fee) |
| [Refunds in the admin panel](#feature-refunds-in-the-admin-panel) |
| [Rotation Constants](#feature-rotation-constants) |
| [Clinical Coordinator management](#feature-clinical-coordinator-management) |
| [Students filter](#feature-students-filter) |
| [Rotation hours API](#feature-rotation-hours-api) |
| [Access control](#feature-access-control) |

### Feature: Student registration and login

```gherkin
Feature: Student registration and login
  As a nursing student
  I want to create an account and log in
  So that I can find a preceptor for my clinical rotation

  Background:
    Given I am on the sign-up page

  Scenario: Student signs up and logs in after email verification
    When I select the "Student" role
    And I fill in all required fields with valid data
    And I submit the sign-up form
    Then I see the message "Check your email to verify your account"
    And I receive a verification email
    When I open the verification link
    And I log in with my email and password
    Then I see the student dashboard

  Scenario: Student cannot log in before email verification
    Given I have signed up but have not verified my email
    When I log in with my email and password
    Then I see a message that my email is not verified
    And I stay on the login page

  Scenario: Student cannot sign up with an email that already exists
    Given a user with the email "existing.student@test.example" already exists
    When I sign up as a student with the email "existing.student@test.example"
    Then I see the error "A user with this email already exists"
    And no new account is created

  Scenario: Login with a wrong password shows a generic error
    Given I am a registered student
    When I log in with my email and a wrong password
    Then I see the error "Invalid email or password"
    And I am not logged in
```

---

### Feature: Rotation request flow

```gherkin
Feature: Rotation request flow
  As a student
  I want to request a rotation with a preceptor and pay for it
  So that my clinical rotation is confirmed

  Background:
    Given a student with a complete profile and approved documents
    And a preceptor with matching specialty, available dates and free capacity

  Scenario: Preceptor accepts the request and the student pays
    When the student sends a rotation request to the preceptor
    Then the request status is "Pending"
    And the preceptor is notified about the new request
    When the preceptor accepts the request
    Then the student is notified that the request was accepted
    When the student pays with a valid card
    Then the rotation status is "Confirmed" for the student, the preceptor and the CC
    And the student receives a payment receipt

  Scenario: Preceptor declines the request
    Given the student has sent a rotation request to the preceptor
    When the preceptor declines the request
    Then the request status is "Declined"
    And the student is notified that the request was declined
    And the student is not charged

  Scenario: Student cannot pay twice for the same rotation
    Given the student has paid for an accepted rotation
    When the student opens the payment page again
    Then no second payment is possible for this rotation
```

---

### Feature: Last-minute fee

```gherkin
Feature: Last-minute fee
  As the business
  I want to charge a last-minute fee for rotations that start soon
  So that urgent placements are priced correctly

  # Rule (assumption): the fee is added when the rotation starts
  # in fewer than "lastMinuteFeeDaysAmount" days from the payment date.

  Background:
    Given the Rotation Constants are:
      | lastMinuteFee | lastMinuteFeeDaysAmount |
      | 100           | 14                      |

  Scenario Outline: Last-minute fee depends on the rotation start date
    Given the preceptor accepted a rotation that starts in <days> days
    When the student opens the payment page
    Then the last-minute fee is <fee>

    Examples:
      | days | fee         |
      | 30   | not added   |
      | 15   | not added   |
      | 14   | not added   |
      | 13   | added (100) |
      | 1    | added (100) |
```

---

### Feature: Refunds in the admin panel

```gherkin
Feature: Refunds in the admin panel
  As a Clinical Coordinator
  I want to see the correct refund information
  So that I can answer student questions about payments

  Background:
    Given I am logged in to the admin panel as a CC
    And a rotation was paid in full with the amount 1000

  Scenario: Partial refund is shown as partial
    Given 300 was refunded in the payment provider
    When I open the payment details of the rotation
    Then the payment status is "Partially refunded"
    And the refunded amount is 300
    And the remaining amount is 700

  Scenario: Full refund is shown as full
    Given 1000 was refunded in the payment provider
    When I open the payment details of the rotation
    Then the payment status is "Refunded"
    And the refunded amount is 1000
```

---

### Feature: Rotation Constants

```gherkin
Feature: Rotation Constants
  As an admin
  I want to edit the rotation fees inline
  So that prices are always up to date

  Background:
    Given I am logged in as an admin with access to the Rotation Constants page
    And "Custom Search Payment One" is 150

  Scenario: Edited value is saved
    When I inline-edit "Custom Search Payment One" to 200
    And I save the change
    Then I see a success message
    When I reload the page
    Then "Custom Search Payment One" is 200

  Scenario: Invalid value is not saved
    When I inline-edit "Custom Search Payment One" to -50
    And I save the change
    Then I see a validation error
    And I do not see a success message
    And "Custom Search Payment One" stays 150 after reload
```

---

### Feature: Clinical Coordinator management

```gherkin
Feature: Clinical Coordinator management
  As an admin
  I want to add new Clinical Coordinators
  So that they can manage students and rotations

  Background:
    Given I am logged in to the admin panel
    And I open the Clinical Coordinators section

  Scenario: CC is created with a new email
    When I add a new CC with all required fields and the email "new.cc@test.example"
    And I click "Save"
    Then the new CC appears in the list

  Scenario: CC is not created with an existing email
    Given a user with the email "existing.cc@test.example" already exists
    When I add a new CC with all required fields and the email "existing.cc@test.example"
    And I click "Save"
    Then I see the error "The user with the provided email already exists (EMAIL_EXISTS)."
    And the CC is not created
```

---

### Feature: Students filter

```gherkin
Feature: Students filter
  As a Clinical Coordinator
  I want to filter students by Program of Study
  So that I can quickly find the students I work with

  Background:
    Given I am logged in to the admin panel as a CC
    And I am on the Students tab

  Scenario Outline: Filter by one or more Programs of Study
    When I select the Programs of Study "<programs>"
    Then the page loads without errors
    And only students from "<programs>" are listed

    Examples:
      | programs           |
      | FNP                |
      | FNP, PMHNP         |
      | FNP, PMHNP, AGPCNP |
```

---

### Feature: Rotation hours API

```gherkin
Feature: Rotation hours API
  As an API client
  I want invalid hours to be rejected
  So that rotations always have valid data

  Background:
    Given I have a valid admin token
    And a new rotation exists

  Scenario Outline: Hours value validation
    When I send POST "/api/v1/rotations/{id}/hours" with hours <hours>
    Then the response status is <status>
    And the hours value is <saved>

    Examples:
      | hours | status | saved     |
      | 120   | 201    | saved     |
      | 1     | 201    | saved     |
      | 0     | 400    | not saved |
      | -40   | 400    | not saved |
      | "abc" | 400    | not saved |
      | null  | 400    | not saved |
```

---

### Feature: Access control

```gherkin
Feature: Access control
  As the platform
  I want each role to see only its own data and pages
  So that personal data is protected

  Scenario: Student cannot open the admin panel
    Given I am logged in as a student
    When I open the Rotation Constants page URL directly
    Then access is denied
    And no admin data is loaded

  Scenario: Student cannot see another student's rotation
    Given I am logged in as Student A
    And Student B has a rotation
    When I open Student B's rotation by its ID
    Then access is denied
    And no data of Student B is shown
```
