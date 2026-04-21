# Challenge Review Comments

Các comment từ reviewer (minhnbwork97) trên PR cập nhật tính năng challenge cho `riz-app-v2` (phần backend `riz-be`).

## Tóm tắt hành động cần làm

- Bổ sung cron job kiểm tra challenge đã end, sau đó cập nhật các logic liên quan.
- Backend sẽ không xử lý parse date và timezone, chỉ trả về đúng chuẩn ISO, timezone UTC+0. Frontend lo phần hiển thị.
- Kiểm tra usage của 2 trường `approachNote` và `themeResponse`, nếu không dùng thì xóa hẳn trong DB.

---

## Chi tiết comment

### 1. `apps/nest/libs/challenge/src/constants/writable-status.ts`

> Trường hợp status `ENDED`, đúng là không nên trực tiếp xử lý bằng api update, nhưng cần 1 cron job (chạy mỗi ngày hoặc mỗi giờ) để check xem 1 challenge đã end hay chưa và update lại trạng thái thành `ENDED`.

**Action**: Thêm cron job tự động chuyển status sang `ENDED` khi `endsAt < now` (chạy định kỳ mỗi giờ hoặc mỗi ngày).

---

### 2. `apps/nest/libs/challenge/src/dtos/get-challenges.dto.ts`

Context: enum `ChallengePhase` hiện đang derive `Ended` từ `status=ACTIVE AND endsAt < now`.

```ts
export enum ChallengePhase {
  Upcoming = 'upcoming', // status=ACTIVE AND startsAt > now
  Ongoing = 'ongoing',   // status=ACTIVE AND startsAt <= now <= endsAt
  Ended = 'ended',       // status=ACTIVE AND endsAt < now
}
```

> Sau khi có cron job đổi trạng thái `ENDED` thì điều kiện chỗ này phải đổi thành check `status = ENDED`.

**Action**: Sau khi có cron job, đổi điều kiện `Ended` thành `status = ENDED` (thay vì derive từ `endsAt < now`).

---

### 3. `apps/nest/libs/challenge/src/helpers/challenge-date.helper.ts`

Context: file đang hard-code `const BUSINESS_TZ_OFFSET = '+07:00'` để round-trip timezone về business TZ.

> Phần timezone không nên fix cứng, ở backend thì cần trả về đúng chuẩn ISO (tức timezone UTC +0), còn ở FE khi hiển thị ra sẽ lấy timezone của thiết bị để hiển thị chính xác.

**Action**:
- Bỏ hard-code `BUSINESS_TZ_OFFSET = '+07:00'`.
- Backend trả về ISO string ở UTC (timezone `+00:00`).
- Frontend (app v2 / admin FE) tự xử lý hiển thị theo timezone của thiết bị.

---

### 4. `apps/nest/libs/challenge/src/entities/challenge.entity.ts`

Context: 2 trường `@Expose()` (liên quan `approachNote` và `themeResponse` theo summary) đã bị xóa khỏi entity.

> sao chỗ này lại xóa đi nhỉ, nếu 2 trường này không dùng thì cần xóa cả trong schema.

**Action**: Kiểm tra lại usage của `approachNote` và `themeResponse`:
- Nếu không dùng: xóa hẳn trong `prisma schema` + migration.
- Nếu vẫn dùng: giữ lại `@Expose()` trong entity.
