# Power Automate Flow Build Guide

## AM/RM master used
This app has 2 AM names and 20 RM names from the fresh file.

## 1. Upload workbook
Upload `RM_Field_Status_Template.xlsx` to OneDrive for Business or SharePoint.

## 2. Trigger
Create a cloud flow with **When an HTTP request is received**.

Paste `power-automate-request-schema.json` into **Request Body JSON Schema**.

## 3. Secret check
Add a Condition:

```text
OR
triggerOutputs()?['headers']?['x-rm-secret'] equals CHANGE_THIS_SHARED_SECRET
triggerBody()?['secretKey'] equals CHANGE_THIS_SHARED_SECRET
```

If false, return Response status `401` with body:

```json
{ "error": "Unauthorized" }
```

## 4. Excel Add Row
Action: **Excel Online (Business) > Add a row into a table**

Table: `StatusLog`

Map fields:

- `SubmitDate` = `submitDate`
- `SubmitDateText` = `submitDateText`
- `AMName` = `amName`
- `RMName` = `rmName`
- `Status` = `status`
- `OtherTaskName` = `otherTaskName`
- `SubmittedAt` = `submittedAt`
- `Source` = `source`

## 5. Run Office Script
Action: **Excel Online (Business) > Run script**

Script: `office-script-generate-daily-summary.ts`

Map script parameters:

- `submitDate` = `submitDate`
- `submitDateText` = `submitDateText`

The script refreshes `DailySummary` in File1-style format:

```text
Date : 6th July'26
ZM Name | Active RM Count | Listing | Maintenance | Others Task | Leave
```

## 6. Response
Return status `200` with body:

```json
{ "message": "Status submitted successfully" }
```

## 7. Connect HTML app
Copy the HTTP POST URL from the trigger and paste into `config.js` as `powerAutomateUrl`.
