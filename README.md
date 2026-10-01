# Script-Controlled Access Control List (ACL) for Institution Details

A ServiceNow project that uses **script-controlled ACLs** to restrict who can view, create, edit and delete records in a custom table. Only users with the right roles can see records where the **Branch** field is **EEE**, while administrators keep full access.

**Team ID:** SWTID-2026-3594

## Team

| Name | Role |
| --- | --- |
| Aaruth K S | Team Leader |
| A Agasi Idhaya | Team Member |
| Ahilan S | Team Member |
| Ashok Kumar K Y | Team Member |
| Ashrafulla S | Team Member |

## Overview

The project shows how user roles and record field values are evaluated together to make access decisions. Four ACLs, one per operation, protect the `u_institution_details` table across forms, lists and Service Portal interfaces.

| Role | ACL granted | What the user can do |
| --- | --- | --- |
| `bb1` | Read (script-controlled, condition: Branch is EEE) | View EEE branch records only |
| `bb2` | Create | Use the **New** button to add records |
| `bb3` | Write | Edit EEE branch records |
| `bb4` | Delete | Delete EEE branch records |
| `admin` | Full access | View all records regardless of branch |

Users without any of these roles cannot view any records.

## Prerequisites

- A ServiceNow instance (for example, a personal developer instance)
- An account with `admin` access
- The `security_admin` role, elevated when creating ACLs
- Estimated time: about 3 hours

## Setup

### 1. Users and roles

1. Log in as admin.
2. Go to **User Administration → Users → New** and create a test user:
   - User ID: `EEE User`
   - First name: `EEE`, Last name: `User`
   - Email: `eeeuser@gmail.com`
3. Go to **User Administration → Roles → New** and create the roles `bb1`, `bb2`, `bb3` and `bb4`.
4. Assign all four roles to the EEE User.

### 2. Custom table

1. Go to **System Definition → Tables → New**.
   - Label: `Institution Details`
   - Name: `u_institution_details`
   - Extends: none
2. Add these fields:

| Field | Type |
| --- | --- |
| Student Roll Number | Auto Number |
| Student Name | Reference (User) |
| Faculty Name | Reference (User) |
| Branch | Choice (ECE, EEE, CSE) |
| Email | String |
| Phone Number | String |
| Description | Multi-line String |

3. Create several records with different Branch values (ECE, EEE, CSE).

### 3. Read ACL

1. Go to **System Security → Access Control (ACL)** and elevate to `security_admin`.
2. Click **New** and enter:
   - Type: `record`
   - Operation: `read`
   - Name: `u_institution_details`
   - Active: `true`
   - Advanced: `true`
3. In **Requires role**, add `bb1`.
4. Add the data condition **Branch is EEE**.
5. Add this script and submit:

```javascript
(function () {
    // Allow admin users full access
    if (gs.hasRole('admin')) {
        return true;
    }

    // Allow only users with bb1 to see EEE records
    if (gs.hasRole('bb1')) {
        return true;
    }

    // Deny access for all others
    return false;
})();
```

### 4. Create, Write and Delete ACLs

Create three more ACLs on `u_institution_details` (Active: `true`) with no data condition:

| Operation | Required role |
| --- | --- |
| Create | `bb2` |
| Write | `bb3` |
| Delete | `bb4` |

## Verification

Impersonate each user and open the list by entering `u_institution_details.list` in the All menu filter.

| User | Expected result |
| --- | --- |
| EEE User with `bb1` | Sees only EEE branch records |
| User without the roles | Sees no records |
| EEE User with `bb1` + `bb2` | Sees EEE records and the **New** button |
| EEE User with `bb1` + `bb2` + `bb3` | Can also edit EEE records |
| EEE User with `bb1` + `bb2` + `bb3` + `bb4` | Can also delete EEE records |
| Admin | Sees all records regardless of branch |

## Project Design Phase-II Deliverables

- **Data Flow Diagram & User Stories**: Level 0 DFD and 11 user stories
- **Solution Requirements**: 7 functional and 6 non-functional requirements

## Outcome

Completing this lab gives a working understanding of how script-controlled Read, Write, Create and Delete ACLs enforce record-level security. It also shows how roles and field values together decide access, protecting sensitive data from unauthorized viewing, modification, creation and deletion.
