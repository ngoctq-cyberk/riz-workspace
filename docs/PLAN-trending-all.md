# PLAN: Trending "All" Management for Architects

## Overview

**What:** Thêm quản lý trending cho tab "All" trong category Architects trên web admin.

**Why:**

- App mobile hiển thị feed mặc định khi user mở app là "All" (gộp content từ 3 subcategories)
- App KHÔNG hiện tab "All" trên UI, chỉ có 3 tabs: Living Room, Kitchen, Bedroom
- Admin cần có khả năng curate riêng danh sách projects cho feed "All" này
- "All" cần có version management riêng biệt (tạo, duplicate, activate) giống các subcategories khác

**Decision:** Sử dụng `subcategoryId = NULL` để đại diện cho "All". Backend đã hỗ trợ pattern này sẵn.

---

## Project Type

**WEB** - Frontend Admin Dashboard + Backend API

---

## Success Criteria

| #   | Criteria                                                               | Verification                                            |
| --- | ---------------------------------------------------------------------- | ------------------------------------------------------- |
| 1   | Admin nhìn thấy card "All" trong Architects section                    | UI hiển thị 4 cards: All, Living Room, Kitchen, Bedroom |
| 2   | Click "All" card navigate đến `/trending/architects`                   | Route hoạt động đúng                                    |
| 3   | Tại `/trending/architects`, admin có thể tạo/manage versions cho "All" | CRUD operations hoạt động                               |
| 4   | Stats cho "All" hiển thị đúng (inTrendingCount, totalCount, listCount) | API trả về stats riêng cho All                          |
| 5   | Có thể activate version cho "All"                                      | Settings key được lưu đúng                              |

---

## Tech Stack

| Layer    | Technology                      | Rationale        |
| -------- | ------------------------------- | ---------------- |
| Frontend | React + TanStack Router + Query | Existing stack   |
| Backend  | NestJS + Prisma                 | Existing stack   |
| State    | React Query                     | Existing pattern |

---

## Approach

### Backend

- `subcategoryId = null` trong `TrendingList` table = "All" list
- Settings key: `trending.category.{categoryId}.active_list_id` (không có subcategory suffix)
- `getStats()` cần update để trả về stats cho "All" (category-level only)

### Frontend

- Thêm card "All" vào `trending-stats-grid.tsx`
- Card "All" click → navigate đến `/trending/architects` (không có subcategory param)
- Routes đã có sẵn: `$category.index.tsx` và `$category.versions.$versionId.tsx`

---

## File Structure (Changes)

```
riz-admin-fe/src/
├── widgets/trending/trending-stats-grid/
│   └── ui/trending-stats-grid.tsx          # UPDATE: Add "All" card
│
├── screens/
│   └── (existing screens work as-is)       # NO CHANGE
│
├── routes/_authenticated/trending/
│   ├── $category.index.tsx                 # VERIFY: Works for "All"
│   └── $category.versions.$versionId.tsx   # VERIFY: Works for "All"

riz-be/apps/nest/libs/project/src/
├── admin-trending.service.ts               # UPDATE: getStats() for "All"
└── (other files work as-is)                # NO CHANGE
```

---

## Task Breakdown

### Phase 1: Backend - Stats cho "All"

#### Task 1.1: Update `getStats()` để return stats cho "All" (subcategory = null)

| Field            | Value                   |
| ---------------- | ----------------------- |
| **Agent**        | `backend-specialist`    |
| **Skill**        | `nodejs-best-practices` |
| **Priority**     | P0                      |
| **Dependencies** | None                    |
| **Estimated**    | 10 min                  |

**INPUT:**

- File: `riz-be/apps/nest/libs/project/src/admin-trending.service.ts`
- Method: `getStats()` (lines 588-644)

**CURRENT BEHAVIOR:**

```typescript
// Line 631-640: Chỉ xử lý khi có subcategories
if (cat.subcategories.length === 0) {
  await processCategory(cat.id);
} else {
  for (const subcat of cat.subcategories) {
    // ...only loops through subcategories
  }
}
```

**REQUIRED CHANGE:**
Khi category có subcategories (như Architects), CŨNG cần gọi `processCategory(cat.id)` để lấy stats cho "All" (category-level, no subcategory).

```typescript
if (cat.subcategories.length === 0) {
  await processCategory(cat.id);
} else {
  // NEW: Add stats for "All" (category-level without subcategory)
  await processCategory(cat.id, undefined, cat.slug.toLowerCase(), undefined);

  for (const subcat of cat.subcategories) {
    if (
      subcategory &&
      subcat.slug.toUpperCase() !== subcategory.toUpperCase()
    ) {
      continue;
    }
    await processCategory(cat.id, subcat.id, cat.slug, subcat.slug);
  }
}
```

**OUTPUT:**
API `GET /admin/trending/stats` returns:

```json
[
  { "category": "architects", "subcategory": null, "inTrendingCount": 5, "totalCount": 100, "listCount": 2 },
  { "category": "architects", "subcategory": "living_room", ... },
  { "category": "architects", "subcategory": "kitchen", ... },
  { "category": "architects", "subcategory": "bedroom", ... }
]
```

**VERIFY:**

```bash
curl http://localhost:3001/admin/trending/stats
# Expect: 4 entries for architects (1 "All" + 3 subcategories)
```

---

### Phase 2: Frontend - UI cho "All"

#### Task 2.1: Thêm card "All" vào `trending-stats-grid.tsx`

| Field            | Value                               |
| ---------------- | ----------------------------------- |
| **Agent**        | `frontend-specialist`               |
| **Skill**        | `react-patterns`, `frontend-design` |
| **Priority**     | P1                                  |
| **Dependencies** | Task 1.1                            |
| **Estimated**    | 15 min                              |

**INPUT:**

- File: `riz-admin-fe/src/widgets/trending/trending-stats-grid/ui/trending-stats-grid.tsx`

**REQUIRED CHANGES:**

1. **Add icon import:**

```typescript
import { IconStack3 } from "@tabler/icons-react";
```

2. **Add "All" config:**

```typescript
// Add after SUBCATEGORY_CONFIG
const ALL_CONFIG = {
  title: "All",
  icon: <IconStack3 className="h-5 w-5" />,
  color: "bg-emerald-500/10 text-emerald-500",
} as const;
```

3. **Add helper to find "All" stats:**

```typescript
// Add function to find "All" stats for a category
function findAllStats(
  displayStats: ReadonlyArray<TrendingStats>,
  category: string,
): TrendingStats | undefined {
  return displayStats.find(
    (s) =>
      s.category.toLowerCase() === category.toLowerCase() && !s.subcategory, // null/undefined = "All"
  );
}
```

4. **Update Architects section UI (lines 137-168):**

```tsx
{
  /* Architects Section with Subcategories */
}
<section className="space-y-4">
  <div className="flex items-center gap-2">
    <IconBuilding className="h-5 w-5 text-blue-500" />
    <h2 className="text-lg font-semibold">Architects</h2>
    <Badge variant="outline" className="ml-2">
      {architectsTotalStats.inTrendingCount} in trending
    </Badge>
  </div>

  <div className="grid gap-4 md:grid-cols-4">
    {/* NEW: "All" card first */}
    <CategoryCard
      key="architects-all"
      category="architects"
      title={ALL_CONFIG.title}
      icon={ALL_CONFIG.icon}
      color={ALL_CONFIG.color}
      stats={findAllStats(displayStats, "architects")}
      isLoading={isLoading}
    />

    {/* Existing subcategory cards */}
    {CATEGORY_SUBCATEGORY_MAP.architects.map((subcategory) => {
      const config = SUBCATEGORY_CONFIG[subcategory];
      const subcategoryStats = findStats(
        displayStats,
        "architects",
        subcategory,
      );
      return (
        <CategoryCard
          key={subcategory}
          category="architects"
          subcategory={subcategory}
          title={config.title}
          icon={config.icon}
          color={config.color}
          stats={subcategoryStats}
          isLoading={isLoading}
          isSubcategory
        />
      );
    })}
  </div>
</section>;
```

**OUTPUT:**

- UI hiển thị 4 cards trong Architects section: All, Living Room, Kitchen, Bedroom
- Grid layout: `md:grid-cols-4` (thay vì 3)

**VERIFY:**

1. Mở browser tại `/trending`
2. Thấy 4 cards trong Architects section
3. Card "All" có màu emerald, icon stack
4. Stats hiển thị đúng cho "All"

---

#### Task 2.2: Verify CategoryCard navigation cho "All"

| Field            | Value                 |
| ---------------- | --------------------- |
| **Agent**        | `frontend-specialist` |
| **Skill**        | `react-patterns`      |
| **Priority**     | P1                    |
| **Dependencies** | Task 2.1              |
| **Estimated**    | 5 min                 |

**INPUT:**

- File: `riz-admin-fe/src/widgets/trending/trending-stats-grid/ui/category-card.tsx`

**CHECK:**

- Khi `subcategory` prop là `undefined`, link phải navigate đến `/trending/${category}`
- Không được navigate đến `/trending/${category}/undefined`

**VERIFY:**

1. Click vào card "All"
2. URL phải là `/trending/architects` (không có `/undefined`)
3. Page hiển thị danh sách versions cho "All"

---

#### Task 2.3: Verify Route handlers cho "All"

| Field            | Value                 |
| ---------------- | --------------------- |
| **Agent**        | `frontend-specialist` |
| **Skill**        | `react-patterns`      |
| **Priority**     | P1                    |
| **Dependencies** | Task 2.2              |
| **Estimated**    | 5 min                 |

**INPUT:**

- Files:
  - `riz-admin-fe/src/routes/_authenticated/trending/$category.index.tsx`
  - `riz-admin-fe/src/routes/_authenticated/trending/$category.versions.$versionId.tsx`

**CHECK:**

- `$category.index.tsx`: Renders `TrendingVersionListPage` với `categorySlug` và KHÔNG có `subcategorySlug`
- `$category.versions.$versionId.tsx`: Renders `TrendingCategoryPage` với `categorySlug` và `versionId`, KHÔNG có `subcategorySlug`

**VERIFY:**

1. Navigate to `/trending/architects`
2. Page loads without errors
3. Create a new version → navigates to `/trending/architects/versions/{id}`
4. Version detail page loads correctly

---

### Phase 3: Integration Testing

#### Task 3.1: End-to-End Flow Test

| Field            | Value              |
| ---------------- | ------------------ |
| **Agent**        | `tester`           |
| **Skill**        | `testing-patterns` |
| **Priority**     | P2                 |
| **Dependencies** | Task 2.3           |
| **Estimated**    | 10 min             |

**VERIFY FLOW:**

1. **Stats page:**
   - [ ] "All" card visible với correct stats
   - [ ] Click "All" → navigates to `/trending/architects`

2. **Version list page (`/trending/architects`):**
   - [ ] Shows versions for "All" (subcategoryId = null)
   - [ ] Can create new version
   - [ ] Can duplicate version
   - [ ] Can delete version

3. **Version detail page (`/trending/architects/versions/{id}`):**
   - [ ] Shows items in the version
   - [ ] Can add/remove projects
   - [ ] Can reorder projects
   - [ ] Can activate version

4. **Mobile API test (optional):**
   ```bash
   # When "All" version is activated, mobile should get the feed
   curl http://localhost:3001/trending/feed?category=architects
   ```

---

## Phase X: Final Verification

### Checklist

- [x] **Lint:** `npm run lint` passes in both FE and BE
- [x] **TypeScript:** `npx tsc --noEmit` passes
- [ ] **Build:** `npm run build` passes
- [ ] **Manual Test:** All success criteria verified

### Success Verification

| #   | Criteria                                   | Status                    |
| --- | ------------------------------------------ | ------------------------- |
| 1   | Card "All" visible in UI                   | ✅ Implemented            |
| 2   | Navigation to `/trending/architects` works | ✅ Verified (code review) |
| 3   | CRUD operations for "All" versions work    | ⏳ Pending manual test    |
| 4   | Stats API returns "All" stats              | ✅ Implemented            |
| 5   | Activate version for "All" works           | ⏳ Pending manual test    |

---

## Risks & Mitigations

| Risk                                                         | Probability | Impact | Mitigation                      |
| ------------------------------------------------------------ | ----------- | ------ | ------------------------------- |
| CategoryCard generates wrong link when subcategory=undefined | Medium      | High   | Task 2.2 verification           |
| Mobile app not handling feed correctly                       | Low         | Medium | Keep same API response format   |
| Stats counting incorrect for "All"                           | Medium      | Low    | Task 1.1 verification with curl |

---

## Notes

- **BE đã hỗ trợ:** `validateCategorySubcategory()`, `getSettingsKey()`, `getLists()` đã handle case `subcategoryId = undefined`
- **Routes đã có:** `$category.index.tsx` và `$category.versions.$versionId.tsx` đã tồn tại
- **Minimal changes:** Chỉ cần update `getStats()` ở BE và `trending-stats-grid.tsx` ở FE
