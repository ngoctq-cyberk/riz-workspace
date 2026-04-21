# E2E Test Plan: Challenge CRUD

## Overview
Test plan cho tính năng **Thêm, Sửa, Xóa Challenge** trên Admin Web.

**Base URL**: `http://localhost:5173`  
**Prerequisite**: Đã đăng nhập vào admin panel

---

## Test Data

```
Title: "E2E Test Challenge - [timestamp]"
Description: "This is an automated E2E test challenge"
Objective: "Test the challenge creation flow"
Requirements: "Submit any creative work"
Rules: "Be creative and original"
Start Date: [tomorrow]
End Date: [next week]
Feed Category: "Architects" (hoặc category có sẵn)
Award Category: "Best Design"
```

---

## TC-01: View Challenge List

**Steps:**
1. Navigate to `/challenges`
2. Verify page loads with title "Challenges"
3. Verify "Create Challenge" button is visible
4. Verify table headers: Cover, Title, Status, Category, Start Date, End Date, Submissions, Members
5. Verify search input with placeholder "Filter by title..."

**Expected:**
- Challenge list table hiển thị
- Pagination hiển thị nếu có nhiều challenges
- Có thể filter by title

---

## TC-02: Create Challenge - Happy Path

**Steps:**
1. Navigate to `/challenges`
2. Click button "Create Challenge"
3. Verify redirected to `/challenges/create`
4. Fill form:
   - Title: "E2E Test Challenge - [timestamp]"
   - Description: "This is an automated E2E test challenge"
   - Objective: "Test the challenge creation flow"
   - Requirements: "Submit any creative work"
   - Rules: "Be creative and original"
   - Start Date: chọn ngày mai
   - End Date: chọn tuần sau
   - Feed Category: chọn bất kỳ option nào (e.g., "Architects")
   - Award Categories: nhập "Best Design"
5. Click button "Create Challenge"
6. Wait for success toast/redirect

**Expected:**
- Redirect to challenge detail page `/challenges/[id]`
- Challenge hiển thị với đúng thông tin đã nhập
- Status là "Draft"

**Verify on Detail Page:**
- Title matches
- Description matches
- Start/End dates match
- Category matches
- Award category "Best Design" hiển thị

---

## TC-03: Create Challenge - Validation Errors

**Steps:**
1. Navigate to `/challenges/create`
2. Click "Create Challenge" without filling any field
3. Verify validation errors appear:
   - "Title is required"
   - "Description is required"
   - "Objective is required"
   - "Requirements are required"
   - "Rules are required"
   - "Start date is required"
   - "End date is required"
   - "Please select a feed category"

**Expected:**
- Form không submit
- Error messages hiển thị dưới mỗi field

---

## TC-04: Create Challenge - End Datetime Validation

**Steps:**
1. Navigate to `/challenges/create`
2. Fill all required fields
3. Set Start datetime: 2026-04-20T10:00
4. Set End datetime: 2026-04-15T10:00 (before start)
5. Click "Create Challenge"

**Expected:**
- Validation error: "End date/time must be later than start date/time"
- Form không submit
- Datetime submitted lên BE ở dạng ISO 8601 UTC (`2026-04-20T03:00:00.000Z` với admin locale +07:00)

---

## TC-05: Edit Challenge - Happy Path

**Steps:**
1. Navigate to `/challenges`
2. Find the challenge created in TC-02 (hoặc bất kỳ challenge nào)
3. Click dropdown menu (icon 3 dots) on that row
4. Click "Edit"
5. Verify redirected to `/challenges/[id]/edit`
6. Verify form pre-filled với existing data
7. Update Title: append " - Updated"
8. Update Description: append " (Updated via E2E test)"
9. Click "Save Changes"

**Expected:**
- Redirect to challenge detail page
- Title shows updated value
- Description shows updated value
- Toast success hiển thị

---

## TC-06: Edit Challenge - Via Detail Page

**Steps:**
1. Navigate to `/challenges`
2. Click on challenge title to go to detail page
3. Click "Edit" button (top right)
4. Verify form loads with existing data
5. Change Feed Category to different value
6. Click "Save Changes"

**Expected:**
- Category updated successfully
- Redirect to detail page
- New category hiển thị

---

## TC-07: Cancel Edit

**Steps:**
1. Navigate to `/challenges/[id]/edit`
2. Modify some fields
3. Click "Cancel" button

**Expected:**
- Navigate back to detail page `/challenges/[id]`
- No changes saved
- Original values preserved

---

## TC-08: Delete Challenge (NOT IMPLEMENTED)

> **Note**: Delete button chưa được implement trong UI. API endpoint tồn tại: `DELETE /admin/challenges/:id`

**TODO:** Khi UI implement delete, test:
1. Navigate to challenge detail hoặc edit page
2. Click "Delete" button
3. Confirm deletion dialog
4. Verify challenge removed from list

---

## TC-09: Search/Filter Challenge List

**Steps:**
1. Navigate to `/challenges`
2. Type test challenge title vào search input
3. Verify table filters to show matching challenges
4. Clear search input
5. Verify all challenges show again

**Expected:**
- Real-time filtering by title
- Case insensitive search

---

## TC-10: Pagination

**Prerequisite:** Có >20 challenges trong system

**Steps:**
1. Navigate to `/challenges`
2. Verify pagination info: "Page 1 of X (Y total)"
3. Click "Next" button
4. Verify page 2 loads
5. Click "Previous" button
6. Verify back to page 1

**Expected:**
- Pagination works correctly
- Next/Previous disabled at boundaries

---

## TC-11: Upload Cover Image

**Steps:**
1. Navigate to `/challenges/create`
2. Click on "Click to upload cover image" area
3. Select an image file
4. Verify image preview shows
5. Click "Remove" button on preview
6. Verify upload area shows again
7. Re-upload image
8. Fill other required fields
9. Submit form

**Expected:**
- Image preview displays after upload
- Remove button clears image
- Challenge created with cover image

---

## TC-12: Add Multiple Award Categories

**Steps:**
1. Navigate to `/challenges/create`
2. Fill required fields
3. In "Award Categories" section:
   - First award: "Best Design"
   - Click "Add Award" button
   - Second award: "Most Creative"
   - Click "Add Award" button
   - Third award: "People's Choice"
4. Submit form

**Expected:**
- All 3 award categories saved
- Detail page shows all 3 awards as badges

---

## TC-13: Remove Award Category

**Steps:**
1. Navigate to `/challenges/create`
2. Add 3 award categories
3. Click trash icon on second award
4. Verify second award removed
5. Submit form

**Expected:**
- Only 2 awards saved
- Cannot remove last remaining award (button hidden when only 1 left)

---

## TC-14: SubCategory (Architects only)

**Steps:**
1. Navigate to `/challenges/create`
2. Select Feed Category: "Architects"
3. Verify SubCategory dropdown is enabled
4. Select a subcategory
5. Change Feed Category to something else
6. Verify SubCategory dropdown is disabled and cleared

**Expected:**
- SubCategory only available for "Architects"
- Switching away clears subcategory selection

---

## UI Element Locators

Dùng các locators sau để tìm elements:

| Element | Locator Strategy |
|---------|------------------|
| Create Challenge button | `button:has-text("Create Challenge")` |
| Title input | `input#title` |
| Description textarea | `textarea#description` |
| Objective textarea | `textarea#objective` |
| Requirements textarea | `textarea#requirements` |
| Rules textarea | `textarea#rules` |
| Start datetime input | `input#startsAt` (type=datetime-local) |
| End datetime input | `input#endsAt` (type=datetime-local) |
| Feed Category dropdown | First Select with placeholder "Select category" |
| Award input | Input with placeholder containing "Award name" |
| Add Award button | `button:has-text("Add Award")` |
| Submit button (create) | `button:has-text("Create Challenge")` |
| Submit button (edit) | `button:has-text("Save Changes")` |
| Cancel button | `button:has-text("Cancel")` |
| Search input | `input[placeholder="Filter by title..."]` |
| Row dropdown | Button with 3 dots icon in table row |
| Edit menu item | Menu item with text "Edit" |
| View menu item | Menu item with text "View" |
| Back button | Ghost button with arrow-left icon |

---

## Notes for Claude Chrome Extension

1. **Wait for loading**: Nhiều pages có loading spinner, đợi cho đến khi spinner biến mất
2. **Toast notifications**: Success/error messages hiển thị ở góc, cần verify
3. **Date inputs**: Dùng format YYYY-MM-DD
4. **File upload**: Click vào dashed border area để trigger file input
5. **Dropdown menu**: Click icon 3 dots trước, rồi mới click menu item
6. **Select component**: Click trigger rồi click option trong dropdown content
