# Asset-Registory-management-System

A Microsoft Power Platform application designed to manage organizational assets, track check-outs and returns, prevent duplicate check-outs, automate due-date reminders, and monitor asset utilization.

---

## 📌 Project Overview

Organizations frequently lend laptops and peripherals such as mice, keyboards, monitors, and headsets to employees. Managing these assets manually can make it difficult to identify available items, track who has borrowed them, and monitor overdue returns.

The **Asset Checkout Registry System** provides a centralized solution for managing asset inventory and tracking the complete check-out and check-in lifecycle.

Built using Microsoft Power Apps, Dataverse, Power Automate, and Power BI, the application streamlines asset management through automated validation, scheduled reminders, and visual analytics.

## 🎯 Objectives

- Maintain a centralized asset register.
- Track asset check-outs, assigned users, and due dates.
- Record asset returns and update availability automatically.
- Prevent an asset from being checked out more than once simultaneously.
- Send automated Microsoft Teams reminders 24 hours before the due date.
- Identify overdue assets and monitor asset utilization.
- Provide analytics to support better asset management decisions.

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Microsoft Power Apps | Canvas application and user interface |
| Microsoft Dataverse | Centralized data storage and relationships |
| Microsoft Power Automate | Check-out validation and automated reminders |
| Microsoft Power BI | Asset utilization and overdue analytics |
| Microsoft Teams | Due-date reminder notifications |
| Power Fx | Application logic and data operations |

## 🏗️ System Architecture

```text
              Employees / IT Administrators
                          |
                          v
                 Microsoft Power Apps
                          |
              +-----------+-----------+
              |                       |
              v                       v
       Microsoft Dataverse      Power Automate
         +-----------+           +-----------+
         |           |           |           |
         |  Asset    |           | Checkout  |
         |  Table    |           | Validation|
         |           |           |           |
         | Checkout  |           | Scheduled |
         |  Table    |           | Reminders |
         +-----------+           +-----------+
                                         |
                                         v
                                  Microsoft Teams
                                  Notifications

              Dataverse Data
                    |
                    v
               Power BI
                    |
                    v
          Utilization & Overdue Reports
```

## 🗂️ Data Model

The application uses two Microsoft Dataverse tables connected through a one-to-many relationship.

### 1. Asset Table

Stores the details and current status of each organizational asset.

| Column | Description |
|---|---|
| Name | Asset name, such as Dell Latitude 5440 |
| Tag | Unique asset tag or barcode |
| Type | Laptop, Mouse, Keyboard, Monitor, Headset, Other |
| Condition | New, Good, Fair, Damaged |
| Status | Available, Checked Out, Under Repair, Retired |

### 2. Checkout Table

Maintains the borrowing and return history for each asset.

| Column | Description |
|---|---|
| Checkout ID | Unique checkout reference |
| Asset | Lookup to the Asset table |
| User | Assigned employee or borrower |
| From | Check-out date and time |
| Due | Expected return date and time |
| ReturnedOn | Actual return date and time |

**Relationship:** One asset can have multiple checkout records over time, but it must never have more than one active checkout at a time.

## 📱 Application Features

### 1. Dashboard

- Displays available and checked-out asset summaries.
- Highlights overdue assets.
- Provides a quick overview of asset availability.

### 2. Check-Out Management

- Displays assets available for borrowing.
- Supports selecting an asset and assigning it to a user.
- Allows the user to specify a due date.
- Creates a checkout record in Dataverse.
- Updates the asset status to `Checked Out`.
- Validates availability to prevent duplicate check-outs.
**Power Fx Implementation**
  
The following code is used in the Check Out button's OnSelect property in Power Apps.

If(
    // Guard: asset must still be available
    galAssets.Selected.Status <> 'Status (Assets)'.Available,
    Notify(
        "This asset is already checked out.",
        NotificationType.Error
    ),
    Patch(
        Checkouts,
        Defaults(Checkouts),
        {
            Asset: galAssets.Selected,
            User: cmbUser.Selected,
            From: Now(),
            Due: dpDue.SelectedDate
        }
    );
    Patch(
        Assets,
        galAssets.Selected,
        {
            Status: 'Status (Assets)'.'Checked Out'
        }
    );
    Notify(
        "Check-out successful.",
        NotificationType.Success
    )
)
### 2. Check-Out Management

The Check-Out screen allows users to select an available asset, assign it to a user, specify a due date, and record the checkout transaction in Microsoft Dataverse.

**Key functionalities:**
- Validates whether the selected asset is available.
- Creates a new checkout record in the `Checkouts` table.
- Records the assigned user, checkout date and time, and due date.
- Updates the asset status to `Checked Out`.
- Displays success or error notifications.

#### Power Fx Implementation

The following code is used in the Check Out button's `OnSelect` property in Power Apps.

```powerfx
If(
    // Guard: asset must still be available
    galAssets.Selected.Status <> 'Status (Assets)'.Available,
    Notify(
        "This asset is already checked out.",
        NotificationType.Error
    ),
    Patch(
        Checkouts,
        Defaults(Checkouts),
        {
            Asset: galAssets.Selected,
            User: cmbUser.Selected,
            From: Now(),
            Due: dpDue.SelectedDate
        }
    );
    Patch(
        Assets,
        galAssets.Selected,
        {
            Status: 'Status (Assets)'.'Checked Out'
        }
    );
    Notify(
        "Check-out successful.",
        NotificationType.Success
    )
)
```

#### Code Explanation

| Component | Description |
|---|---|
| `galAssets.Selected` | Retrieves the selected asset from the gallery. |
| `cmbUser.Selected` | Retrieves the selected user from the combo box. |
| `dpDue.SelectedDate` | Retrieves the selected due date. |
| `Now()` | Records the current date and time of checkout. |
| `Defaults(Checkouts)` | Creates a new checkout record. |
| `Patch()` | Creates or updates records in Dataverse. |
| `Notify()` | Displays success or error messages. |


### 3. Check-In Management

- Identifies assets currently checked out.
- Records the return date and time.
- Updates the asset status to `Available`.
- Preserves checkout history for future reference.
#### Power Fx Implementation

The following code is used in the Check in button's `OnSelect` property in Power Apps.

With(
{ openCo: LookUp(Checkouts, Asset.Tag = galOut.Selected.Tag && 
IsBlank(ReturnedOn)) },
Patch(Checkouts, openCo, { ReturnedOn: Now() });
Patch(Assets, galOut.Selected, { Status: 'Status (Assets)'.Available });
Notify("Asset returned.", NotificationType.Success)
)

### 4. Overdue Tracking

- Identifies assets that have passed their due dates and have not been returned.
- Displays overdue items on the dedicated Overdue screen.
- Supports overdue monitoring through the dashboard.

#### Power Fx Implementation

The following code is used in the Overdue button's `OnSelect` property in Power Apps.

Filter(Checkouts, Text(ThisRecord.'Checkout Status') = "Active", 'past date' < Now())

### 5. Automated Reminders

The Power Automate reminder flow:

1. Runs on a scheduled recurrence.
2. Identifies open checkout records due within the next 24 hours.
3. Sends Microsoft Teams notifications to the assigned users.
4. Includes the asset name, asset tag, and due date in the reminder.

### 6. Asset Analytics

Power BI reporting supports analysis of:

- Asset utilization percentage.
- Current asset status distribution.
- Overdue checkout counts.
- Overdue trends by week or month.

## ⚙️ Business Logic

### Preventing Duplicate Check-Outs

The system uses application-level validation and a Power Automate flow named `Flow_CheckoutAsset` to check for an existing open checkout before creating another record.

If an open checkout exists for the selected asset, the operation is rejected. Otherwise, the checkout record is created and the asset status is updated.

### Identifying Overdue Assets

An asset is considered overdue when its checkout record has no return timestamp and its due date has passed.

Illustrative Power Fx filter:

```powerfx
Filter(
    Checkouts,
    IsBlank(ReturnedOn) && Due < Now()
)
```

### Calculating Asset Utilization

Illustrative Power BI DAX measure:

```dax
Utilisation % =
DIVIDE(
    CALCULATE(
        COUNTROWS(Asset),
        Asset[Status] = "Checked Out"
    ),
    COUNTROWS(Asset)
)
```

### Calculating Overdue Count

```dax
Overdue Count =
CALCULATE(
    COUNTROWS(Checkout),
    ISBLANK(Checkout[ReturnedOn]),
    Checkout[Due] < NOW()
)
```

*Note: Adjust table names, column names, and choice values to match the actual Dataverse and Power BI schema.*

## 🧪 Testing and Validation

The project documentation records six successful acceptance tests.

| Test Case | Expected Outcome | Result |
|---|---|---|
| Check out an available laptop | Checkout record created and asset marked Checked Out | ✅ Pass |
| Check out the same laptop again | Duplicate check-out blocked with an error | ✅ Pass |
| Return a checked-out laptop | Return timestamp recorded and asset marked Available | ✅ Pass |
| Check out an item due within 24 hours | Teams reminder received | ✅ Pass |
| Leave an item past its due date | Item appears on the Overdue screen and dashboard | ✅ Pass |
| Verify analytics | Utilization and overdue figures match Dataverse data | ✅ Pass |

**Acceptance criteria achieved:**
- Simultaneous duplicate check-outs are prevented.
- Due-date reminders are delivered as designed.

## 🧠 Challenges and Learnings

- Designing one-to-many relationships in Microsoft Dataverse.
- Working with lookup columns and related records.
- Using Power Fx `Patch()` operations to update application data.
- Maintaining consistent asset and checkout statuses.
- Implementing server-side validation using Power Automate.
- Scheduling automated reminders and notifying users through Microsoft Teams.
- Developing asset utilization and overdue reporting.

## 🚀 Future Enhancements

- Add an approval workflow for high-value assets.
- Send escalation emails to managers when assets remain overdue for more than three days.
- Introduce QR-code labels for asset identification.
- Integrate AI Builder object detection to suggest asset types from photographs.

*AI Builder photo recognition is a proposed enhancement and is not part of the completed MVP.*

## 📋 Project Scope and Status

| Component | Status |
|---|---|
| Dataverse Asset and Checkout tables | Completed |
| Power Apps canvas application | Completed |
| Check-out validation flow | Completed |
| Automated reminder flow | Completed |
| Asset utilization and overdue analytics | Completed |
| Acceptance testing | Completed |
| AI Builder object detection | Future enhancement |

## 🔮 Conclusion

The Asset Checkout Registry System demonstrates how Microsoft Power Platform can be used to build a practical asset management solution with centralized data, automated workflows, and business intelligence.

The completed MVP supports asset registration, check-out and check-in tracking, duplicate check-out prevention, automated reminders, overdue monitoring, and utilization reporting.

This project showcases practical skills in **Microsoft Power Apps, Dataverse, Power Automate, Power BI, and Power Fx**, along with workflow automation, relational data modeling, and business process improvement.

---

**Developed by:** Shravan  
**Platform:** Microsoft Power Platform  
**Project Status:** Completed  
