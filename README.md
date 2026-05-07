# Classic-Report-Select-UnSelect-All-with-row-status-wise-check-box-enable-disabled-in-Oracle-APEX

This document provides a complete implementation guide for adding "Select / UnSelect All" functionality to a classic report in Oracle APEX using `APEX_ITEM.CHECKBOX` with enhanced features including:
- Automatic disabling of checkboxes for approved records
- Smart "Select All" that excludes disabled checkboxes
- Bulk approval button state management
- Processing selected items to insert/update into another table

---

## 📋 Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Implementation Steps](#implementation-steps)
  - [Step 1: Classic Report SQL Query](#step-1-classic-report-sql-query)
  - [Step 2: Change Select Column Label/Header](#step-2-change-select-column-labelheader)
  - [Step 3: Add JavaScript Code](#step-3-add-javascript-code)
  - [Step 4: Create Bulk Approval Button](#step-4-create-bulk-approval-button)
  - [Step 5: Create PL/SQL Process](#step-5-create-plsql-process)
  - [Step 6: Add Success Message (Optional)](#step-6-add-success-message-optional)
- [Complete Working Example](#complete-working-example)
- [Troubleshooting](#troubleshooting)
- [Demo](#demo)
- [Connect with Me](#connect-with-me)

---

## ✨ Features

✅ **Select / Unselect All** - Single click to select/deselect all checkboxes  
✅ **Disabled Checkbox Handling** - Already approved records are automatically disabled  
✅ **Smart Select All** - Only selects enabled (non-approved) checkboxes  
✅ **Real-time State Management** - Select All checkbox updates dynamically  
✅ **Bulk Approval Protection** - Prevents processing of already approved records  
✅ **Visual Feedback** - User-friendly indicators and messages  
✅ **Region Refresh Safe** - Works after interactive grid refreshes  

---

## 📋 Prerequisites

- Oracle APEX 19.1 or higher
- Oracle Database 11g or higher
- Table with at least these columns:
  - Primary Key column (e.g., `PI_ID`)
  - Status column (e.g., `APPROVAL_STATUS`) 
  - Other required columns

---

## 🚀 Implementation Steps

### Step 1: Classic Report SQL Query

Create a Classic Report with the following SQL query:

```sql
SELECT
    APEX_ITEM.checkbox(
        1,
        A.PI_ID,
        'class="chk_select" data-pi-id="' || A.PI_ID || 
        '" data-status="' || NVL(A.APPROVAL_STATUS, 'PENDING') || 
        '" style="width:18px;height:18px;cursor:pointer;"' ||
        CASE WHEN A.APPROVAL_STATUS = 'APPROVED' THEN ' disabled' ELSE '' END
    ) AS SELECTPI,
    A.PI_NUMBER,
    A.PI_DATE,
    A.SUPPLIER_NAME,
    A.APPROVAL_STATUS
FROM CM_EXPORT_ATTACH_PI A
WHERE A.EXPORT_LC_ID = :P48_EXPORT_LC_ID  -- Your filter condition
ORDER BY A.PI_DATE DESC;
```
## 2. Chenge select column label/header

- In the Column Heading for SELECTPI, insert this HTML:
- column Heading
```
<input type="checkbox" id="chk_select_all" style="width:18px;height:18px;cursor:pointer;" title="Select / Unselect all">

```
---
### 3. Added below javascript code in your Page → Execute when Page Loads section:


// Function to update Select All checkbox state based on enabled checkboxes only
function updateSelectAllState() {
    var totalEnabled = $('.chk_select:not(:disabled)').length;
    var checkedEnabled = $('.chk_select:not(:disabled):checked').length;
    
    if (totalEnabled === 0) {
        $('#chk_select_all').prop('checked', false).prop('disabled', true);
    } else {
        $('#chk_select_all').prop('disabled', false);
        $('#chk_select_all').prop('checked', checkedEnabled === totalEnabled);
    }
}

// Handle Select All click - only affects enabled checkboxes
$(document).on('change', '#chk_select_all', function() {
    var isChecked = this.checked;
    $('.chk_select:not(:disabled)').prop('checked', isChecked).trigger('change');
});

// Update Select All state when individual checkboxes change
$(document).on('change', '.chk_select', function() {
    updateSelectAllState();
});

// Initialize on page load
$(document).ready(function() {
    updateSelectAllState();
});

```

### 4. Create a 'ADD' button and Create a PL/SQL Process


```Plsql process
DECLARE
    V_APPROVER_ID NUMBER;
BEGIN
    SELECT PERSON_ID 
    INTO V_APPROVER_ID
    FROM EMPLOYEE_MASTER
    WHERE UPPER(EMAIL_ADDRESS) = UPPER(:APP_USER)
   --- AND PERSON_ID  = 66159496
    AND ROWNUM=1;

    IF apex_application.g_f01.COUNT > 0 THEN
        FOR i IN 1 .. apex_application.g_f01.COUNT LOOP
        		 APPROVAL_PKG.PROCESS_APPROVAL(
                    P_PRIZE_WINNER_LINE_ID => apex_application.g_f01(i),
                    P_APPROVER_ID => V_APPROVER_ID,
                    P_ACTION => 'APPROVE',
                    P_COMMENTS => 'TEST',--:P14_COMMENTS,
                    P_REASSIGN_TO_ID => NULL
                    );

            

        END LOOP;
    END IF;

    -- Success message
    APEX_APPLICATION.G_PRINT_SUCCESS_MESSAGE :=  'Action processed successfully';


EXCEPTION
    WHEN NO_DATA_FOUND THEN
        APEX_ERROR.ADD_ERROR(
            p_message          => 'Budget Approver not found for user: ' || :APP_USER,
            p_display_location => APEX_ERROR.C_INLINE_IN_NOTIFICATION
        );

    WHEN OTHERS THEN
        APEX_ERROR.ADD_ERROR(
            p_message          => 'Error: ' || SQLERRM,
            p_display_location => APEX_ERROR.C_INLINE_IN_NOTIFICATION
        );

       
END;


```
- Refrsh the terget Region/Report


 # Thank you
 ## Sanjay Sikder

 You can connect with me on [LinkedIn](https://www.linkedin.com/in/sanjay-sikder/)!
