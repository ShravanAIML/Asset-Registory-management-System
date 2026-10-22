
```
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


