# Fix: QueryClient Error - Force Update Feature

## Vấn đề

Khi chạy app, gặp lỗi:
```
ERROR  [Error: No QueryClient set, use QueryClientProvider to set one]
```

## Nguyên nhân

Hook `useAppConfig()` được gọi **bên ngoài** `QueryProvider` trong component tree. React Query hooks phải được gọi **bên trong** component có `QueryClientProvider`.

### Code ban đầu (SAI):

```tsx
export default function RootLayout() {
  // ❌ Hook được gọi TRƯỚC khi QueryProvider được mount
  const { data: appConfig } = useAppConfig();

  return (
    <QueryProvider>
      {/* App content */}
    </QueryProvider>
  );
}
```

## Giải pháp

Tạo wrapper component `AppUpdateProvider` để chứa toàn bộ logic app config và modals. Provider này được đặt **bên trong** `QueryProvider`.

### Các thay đổi:

**1. Tạo `AppUpdateProvider` component mới**

File: `riz-app-v2/components/app-update/AppUpdateProvider.tsx`

- Move toàn bộ logic app config từ `_layout.tsx`
- Bao gồm: `useAppConfig()`, `useForegroundConfigCheck()`, dialog state logic
- Render 3 modals (Maintenance, ForceUpdate, SoftUpdate)
- Render `children` để wrap app content

**2. Update `_layout.tsx`**

```tsx
export default function RootLayout() {
  return (
    <GestureHandlerRootView>
      <LocalizationProvider>
        <QueryProvider>
          {/* ✅ AppUpdateProvider BÊN TRONG QueryProvider */}
          <AppUpdateProvider>
            <BottomSheetModalProvider>
              {/* App content */}
            </BottomSheetModalProvider>
          </AppUpdateProvider>
        </QueryProvider>
      </LocalizationProvider>
    </GestureHandlerRootView>
  );
}
```

**3. Update barrel export**

File: `riz-app-v2/components/app-update/index.ts`

Thêm export cho `AppUpdateProvider`.

## Provider Hierarchy

```
GestureHandlerRootView
└── LocalizationProvider (react-intl)
    └── QueryProvider (TanStack Query)
        └── AppUpdateProvider ✅ CÓ THỂ SỬ DỤNG useQuery
            └── BottomSheetModalProvider
                └── ThemeProvider
                    └── SafeAreaProvider
                        └── App Content

Modals (rendered by AppUpdateProvider):
- MaintenanceModal
- ForceUpdateModal
- SoftUpdateModal
```

## Kết quả

- ✅ Không còn `QueryClient` error
- ✅ `useAppConfig()` hoạt động bình thường
- ✅ Modals render đúng vị trí (root level, after app content)
- ✅ Logic app config được encapsulate trong provider riêng

## Files Modified

1. **Created:** `riz-app-v2/components/app-update/AppUpdateProvider.tsx`
2. **Modified:** `riz-app-v2/app/_layout.tsx` - Remove app config logic, wrap với AppUpdateProvider
3. **Modified:** `riz-app-v2/components/app-update/index.ts` - Export AppUpdateProvider
