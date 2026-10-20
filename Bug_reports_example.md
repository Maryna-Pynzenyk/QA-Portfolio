## Bug Reports

| ID | Title| Priority |
|---|---|---|
| [CPP-BUG-001](#cpp-bug-001) | [Admin Panel – Coordinators] Generic "Not created" error is shown instead of the EMAIL_EXISTS error when creating a CC with an existing email in the Admin panel | Medium |
| [CPP-BUG-002](#cpp-bug-002) | [Payments] Partial refund in Stripe for rotation is shown as a full refund in the Admin panel | High |
| [CPP-BUG-003](#cpp-bug-003) | [Regression] Page fails to load when filtering by two Programs of Study on the Students tab – GraphQL error: programOfStudy Int cannot represent non-integer value | Medium |
| [CPP-BUG-004](#cpp-bug-004) | [Rotation Constants] Inline edit returns Success, but no field values are saved while updating rotation constants in the Admin panel| High |
| [CPP-BUG-005](#cpp-bug-005) | API: `POST /api/v1/rotations/{id}/hours` accepts a negative hours value during rotation creation and returns 201 Created | High |


## CPP-BUG-001

### Title
[Admin Panel – Coordinators] Generic "Not created" error is shown instead of the EMAIL_EXISTS error when creating a CC with an existing email in the Admin panel

### Description
When a CC is created in the admin panel with an email that already belongs to an existing user, the UI shows a generic "Not created" error instead of the specific EMAIL_EXISTS message. The admin can't tell why creation failed or how to fix it.

### Preconditions
- CC is logged in to the admin panel.
- A user with the used email already exists in the system.
- The Admin Panel is open.

### Steps to reproduce
1. Open the Clinical Coordinators section in the admin panel.
2. Click the button to add a new CC.
3. Fill in all required fields.
4. Enter the email of an existing user in the Email field.
5. Click "Save".

### Actual result
A generic "Not created" error is displayed, and the CC is not created.

### Expected result
The error "The user with the provided email already exists (EMAIL_EXISTS)." is displayed, and the CC is not created.

### Environment
- Environment: Staging
- Build_version: xxxxx
- Screen size: 1920x1080
- Browser: Microsoft Edge, Version 148.0.7778.179 (Official Build) (64-bit)
- Platform: Windows 11 Pro 64-bit




## CPP-BUG-002

### Title
[Payments] Partial refund in Stripe for rotation is shown as a full refund in the Admin panel

### Description
When a payment is partially refunded via Stripe, the admin panel incorrectly displays the payment as fully refunded, misrepresenting the actual refund amount to the CC.

### Preconditions
- A student is signed up.
- The wizard is completed.
- A rotation is created.
- A preceptor is reserved.
- The first payment is charged.

### Steps to reproduce
1. Open Stripe test mode.
2. Open the created and paid rotation.
3. Click the [Refund] button.
4. Enter half of the sum.
5. Confirm the refund.
6. Log in to the admin panel as CC.
7. Navigate to the rotation associated with the partially refunded payment.
8. Open the Finances details for that rotation.
4. Observe the refund amount displayed in the admin panel.

### Actual result
The admin panel shows the payment as fully refunded even though Stripe processed only a partial refund.

### Expected result
The admin panel shows the correct partial refund amount, matching the amount refunded in Stripe.

### Environment
- Environment: Stage
- Build_version: xxxxx
- Screen size: 1920x1080
- Browser: Microsoft Edge, Version 148.0.7778.179 (Official Build) (64-bit)
- Platform: Windows 11 Pro 64-bit




## CPP-BUG-003

### Title
[Regression] Page fails to load when filtering by two Programs of Study on the Students tab – GraphQL error: programOfStudy Int cannot represent non-integer value

### Description
When filtering by two Programs of Study on the Students tab page, the page fails to load, and a GraphQL error is returned. The error indicates that the `programOfStudy` array values are being passed as strings instead of integers, causing the query to fail.

### Preconditions
- The CC is logged in to the Admin panel.

### Steps to reproduce
1. Navigate to the Students tab page.
2. Apply a filter selecting two Programs of Study simultaneously.
3. Observe that the page fails to load and an error is returned.

### Actual result
The page fails to load, and the following GraphQL error is returned:

```json
{
    "errors": [
        {
            "message": "Variable '$programOfStudy' got invalid value '10' at 'programOfStudy[1]'; Int cannot represent non-integer value: '10'",
            "locations": [
                {
                    "line": 1,
                    "column": 168
                }
            ]
        }
    ]
}
```

### Root cause
The `programOfStudy` array values are being passed as strings (e.g., `'10'`) instead of integers (`10`) in the GraphQL query. The schema expects an array of `Int` but is receiving an array of `String`. This appears to affect only the second (and possibly subsequent) selected values in the array.

### Expected result
Filtering by two or more Programs of Study should successfully return a filtered list of students matching the selected programs.

### Environment
- Environment: Stage
- Build_version: xxxxx
- Scope: Regression
- Screen size: 1920x1080
- Browser: Microsoft Edge, Version 147.0.7727.117 (Official Build) (64-bit)
- Platform: Windows 11 Pro 64-bit




## CPP-BUG-004

### Title
[Rotation Constants] Inline edit returns Success, but no field values are saved while updating rotation constants in the Admin panel

### Description
On the admin Rotation Constants page, saving any field shows a success message, but the value isn't actually saved. After a reload, the old value comes back.

The frontend sends the right record, field and new value, and the `saveInlineEdit` API replies "Success". Even so, nothing changes on the backend, so the problem is on the server side. One likely cause is that the values are sent as strings (`"200"`) for integer fields and are silently dropped.

### Preconditions
Logged in as an admin with access to the Rotation Constants page.

### Steps to reproduce
1. Open the Rotation Constants page.
2. Inline-edit Custom Search Payment One from 150 to 200.
3. Click Save.
3. Observe the success message.
4. Hard reload the page (cache disabled).

### Actual result
The value reverts to 150 after reload.

- The `saveInlineEdit` mutation returns 200 OK with `status: "Success"` and the message "Inline field has been saved successfully."
- The following `rotationConstants` query returns the unchanged `customSearchPaymentOne: 150`.
- Reproduces for **all fields** on the page: `platformFee`, `platformFeeNoHonorarium`, `serviceFee`, `customSearchPaymentOne`, `customSearchPaymentTwo`, `lastMinuteFee`, `lastMinuteFeeDaysAmount`.

### Technical details

```text
Mutation: saveInlineEdit
Variables:
  contentTypeId: <content-type-id>
  objectUuid: <object-uuid>
  value: {"custom_search_payment_one":"200"}
Response: {"status":"Success","message":"Inline field has been saved successfully."}
```

- The frontend sends the correct record, field and value, so the issue appears to be on the backend.
- The value is sent as a string (`"200"`) for integer fields. It may not be cast/validated, causing the update to be skipped silently while Success is returned.
- The mutation response returns no updated entity, so the client cache cannot refresh from it.

### Impact
Admins cannot change fee constants, and the UI falsely confirms the save.

### Expected result
The value is saved and shows 200 after reload. The `rotationConstants` query returns `customSearchPaymentOne: 200`.

### Environment
- Environment: Stage
- Build_version: xxxxx
- Screen size: 1920x1080
- Browser: Microsoft Edge, Version 152.0.7977.75 (Official Build) (64-bit)
- Platform: Windows 11 Pro 64-bit




## CPP-BUG-005

### Title
API: `POST /api/v1/rotations/{id}/hours` accepts a negative hours value during rotation creation and returns 201 Created

### Preconditions
- A valid access token for an admin user.

### Steps to reproduce (Postman)
1. Send the request:

```http
POST /api/v1/rotations/rot_1001/hours
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "hours": -40
}
```

2. Check the response status and body.
3. Send `GET /api/v1/rotations/rot_1001` and check the `hours` value.

### Actual result
- Step 2: **201 Created**.

```json
{
  "rotationId": "rot_1001",
  "hours": -40
}
```

- Step 3: the rotation is saved with `"hours": -40`.

### Expected result
**400 Bad Request**, and the value is not saved:

```json
{
  "error": "VALIDATION_ERROR",
  "field": "hours",
  "message": "Hours must be a positive number."
}
```

### Environment
- Environment: Stage
- Tool: Postman
