

## 📱 Application Features

### 1. Dashboard

- Displays available and checked-out asset summaries.
- Highlights overdue assets.
- Provides a quick overview of asset availability.

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

### 3. Check-In Management

- Identifies assets currently checked out.
- Records the return date and time.
- Updates the asset status to `Available`.
- Preserves checkout history for future reference.

### 4. Overdue Tracking

- Identifies assets that have passed their due dates and have not been returned.
- Displays overdue items on the dedicated Overdue screen.
- Supports overdue monitoring through the dashboard.

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


