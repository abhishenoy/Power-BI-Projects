# Power BI Projects
This Repo Contains Power Bi Projects

# Sales Report with Dynamic Row-Level Security (Power BI)

A Power BI report that shows sales data with **dynamic row-level security (RLS) based on an employee hierarchy**. Each user signs in and sees only their own data plus the data of everyone who reports to them, directly or indirectly.

## Overview

- Managers see their own sales and their entire team's sales.
- Individual contributors see only their own sales.
- Top-level leadership sees all data.
- Access is driven by the signed-in user's email, so one role covers all users and no manual role assignment per person is needed.

## Data Model

| Table | Purpose |
|---|---|
| `Hierarchy table` | Parent-child employee hierarchy (`EmployeeID`, `ManagerID`, `Name`, `Email`, `Path`) |
| `Sales` | Sales transactions (`SaleID`, `EmployeeID`, `Product`, `Amount`, `OrderDate`, `UnitPrice`) |

**Relationship:** `Hierarchy table[EmployeeID]` (one) → `Sales[EmployeeID]` (many), single direction.

## How the RLS Works

1. **Path column:** a calculated column flattens the hierarchy into a chain from the top manager to each employee (e.g. `1|2|4|7`).
   ```dax
   Path = PATH ( 'Hierarchy table'[EmployeeID], 'Hierarchy table'[ManagerID] )
   ```
2. **Role filter** on `Hierarchy table`:
   ```dax
   PATHCONTAINS (
       'Hierarchy table'[Path],
       LOOKUPVALUE (
           'Hierarchy table'[EmployeeID],
           'Hierarchy table'[Email], USERPRINCIPALNAME ()
       )
   )
   ```
3. The filter flows through the relationship to `Sales`, so the fact table is secured automatically.

The top-level manager's `ManagerID` must be **blank (null)**, not 0 or an empty string.

## Key Measures

- **Total Sales:** sum of `Sales[Amount]`
- **Latest Price:** latest unit price per product
  ```dax
  Latest Price =
  MAXX (
      TOPN ( 1, Sales, Sales[OrderDate], DESC, Sales[SaleID], DESC ),
      Sales[UnitPrice]
  )
  ```

## Testing the Security

In Power BI Desktop, go to **Modeling > View as**, select the role and enter a user's email.

| User | Sees | Total Amount |
|---|---|---|
| anita@company.com | Everyone | All sales |
| rahul@company.com | Self + team | Team total |
| divya@company.com | Self only | Own total |

## Setup

1. Open the `.pbix` file in Power BI Desktop.
2. Replace the sample data with your own source if needed.
3. Publish to the Power BI Service.
4. Go to the semantic model > **Security** and add users or groups to the RLS role.
5. Test with a Viewer account. Workspace Admins, Members and Contributors bypass RLS.

## Tech Used

Power BI, DAX, Power Query, Row-Level Security (RLS)

## Notes

- The sample data is fictional and for demonstration only.
- Make sure the email in the hierarchy table matches the user's sign-in email.
