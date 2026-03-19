# Local Development Guide

Hướng dẫn setup và chạy local development cho dự án RIZ.

---

## Prerequisites

- **Docker Desktop** - Chạy PostgreSQL và Redis
- **Node.js** - v18+
- **pnpm** - Package manager cho Backend
- **bun** - Package manager cho Frontend và Mobile App
- **Xcode** - (Optional) Cho iOS development - cài từ App Store

---

## 1. Khởi động Docker

### Mở Docker Desktop

```bash
open -a Docker
```

### Khởi động containers

```bash
cd riz-be/apps/nest
docker-compose up -d
```

### Kiểm tra containers

```bash
docker ps
```

| Container  | Port             | Mô tả         |
| ---------- | ---------------- | ------------- |
| `postgres` | `localhost:5403` | PostgreSQL 17 |
| `redis`    | `localhost:6849` | Redis Stack   |

---

## 2. Backend (riz-be)

### Cài đặt dependencies

```bash
cd riz-be
pnpm install
```

### Chạy Prisma migrations

```bash
cd riz-be/apps/nest

# Chạy migrations
pnpm prisma

pnpm build

# Seed data (optional - chỉ cần khi DB mới)
DATABASE_URL="postgresql://cyberk:cyberk@localhost:5403/cyberk?schema=public" \
pnpm exec prisma db seed
```

### Khởi động Backend server

```bash
cd riz-be/apps/nest
pnpm start:local:watch
```

| URL                          | Mô tả        |
| ---------------------------- | ------------ |
| `http://localhost:4000`      | API Server   |
| `http://localhost:4000/docs` | Swagger Docs |

---

## 3. Frontend (riz-admin-fe)

### Cài đặt dependencies

```bash
cd riz-admin-fe
bun install
```

### Khởi động Frontend server

```bash
cd riz-admin-fe
bun dev
```

| URL                     | Mô tả       |
| ----------------------- | ----------- |
| `http://localhost:5173` | Admin Panel |

---

## 5. Prisma Studio (Optional)

Xem và quản lý database trực tiếp qua UI.

```bash
cd riz-be/apps/nest
DATABASE_URL="postgresql://cyberk:cyberk@localhost:5403/cyberk?schema=public" \
pnpm exec prisma studio
```

| URL                     | Mô tả         |
| ----------------------- | ------------- |
| `http://localhost:5555` | Prisma Studio |

---

## 6. Tắt Docker

```bash
cd riz-be/apps/nest
docker-compose down
```

---

## Environment Files

### Backend: `riz-be/apps/nest/.env.local`

```env
LOCAL_DEV=true

SUPER_ADMIN_USERNAME=superadmin@gg.com
SUPER_ADMIN_PASSWORD=samplepassword

JWT_SECRET=iu34gt72238r2983jr9jf
JWT_EXPIRES=1d
JWT_REFRESH_SECRET=982gh38th092309rj23
JWT_REFRESH_EXPIRES=365d
VERIFY_CODE_EXPIRE_TIME=86400000

REDIS_URL=redis://:@localhost:6849/1
DATABASE_URL=postgresql://cyberk:cyberk@localhost:5403/cyberk?schema=public

AWS_ACCESS_KEY_ID=<your-aws-key>
AWS_SECRET_ACCESS_KEY=<your-aws-secret>
AWS_REGION=ap-southeast-1
AWS_S3_BUCKET=cyberk-io-blog-dev
AWS_CLOUDFRONT_DOMAIN=d2tpbp6lm18knv.cloudfront.net

SENDGRID=<your-sendgrid-key>
MAIL_FROM=test@radicalinsightzone.com
```

### Frontend: `riz-admin-fe/.env`

```env
VITE_GOOGLE_CLIENT_ID=VITE_GOOGLE_CLIENT_ID
VITE_API_URL=http://localhost:4000
VITE_REFRESH_TOKEN_API_URL=http://localhost:4000/auth/refreshToken
```

---

## Quick Start (All-in-one)

Mở 4 terminals:

**Terminal 1 - Docker:**

```bash
cd riz-be/apps/nest && docker-compose up -d
```

**Terminal 2 - Backend:**

```bash
cd riz-be/apps/nest && pnpm start:local:watch
```

**Terminal 3 - Frontend:**

```bash
cd riz-admin-fe && bun dev
```

**Terminal 4 - Mobile App (Optional):**

```bash
cd riz-app-v2 && bun dev
# Hoặc build lần đầu: npx expo run:ios
```

---

## Troubleshooting

### Docker không chạy

```bash
# Kiểm tra Docker daemon
docker info

# Nếu lỗi, mở Docker Desktop và chờ khởi động
open -a Docker
```

### Prisma migration lỗi

```bash
# Reset database (XÓA TOÀN BỘ DATA)
cd riz-be/apps/nest
DATABASE_URL="postgresql://cyberk:cyberk@localhost:5403/cyberk?schema=public" \
pnpm exec prisma migrate reset --force
```

### Port đang được sử dụng

```bash
# Tìm process đang dùng port 4000
lsof -i :4000

# Kill process
kill -9 <PID>
```

### Frontend không kết nối được Backend

1. Kiểm tra Backend đang chạy: `curl http://localhost:4000/docs`
2. Kiểm tra file `.env` của Frontend có `VITE_API_URL=http://localhost:4000`
3. Restart Frontend sau khi sửa `.env`

---

## 4. Mobile App (riz-app-v2)

### Prerequisites

- **Xcode** - Đã cài đặt đầy đủ từ App Store (cho iOS development)
- **Xcode Command Line Tools** - Đã setup đúng:
  ```bash
  sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
  xcodebuild -version  # Verify Xcode installed correctly
  ```
- **iOS Simulator** - Mở simulator trước khi chạy app:
  ```bash
  open -a Simulator
  ```

### Cài đặt dependencies

```bash
cd riz-app-v2
bun install
```

### Khởi động Mobile App trên iOS Simulator

**Lần đầu tiên (build native app):**

```bash
cd riz-app-v2
npx expo run:ios
```

Lệnh này sẽ:

- Build native iOS app với Expo Dev Client
- Cài CocoaPods dependencies
- Compile Xcode project
- Tự động mở simulator và cài app
- Khởi động Metro bundler

**Các lần sau (sử dụng app đã build):**

```bash
cd riz-app-v2
bun ios
# hoặc
bun dev
```

Nhanh hơn nhiều vì không cần rebuild native code.

### Environment File: `riz-app-v2/.env`

```env
EXPO_PUBLIC_API_URL=http://localhost:4000
EXPO_PUBLIC_REFRESH_TOKEN_API_URL=http://localhost:4000/auth/refreshToken
```

### Development Tools

| Command                     | Mô tả                                                   |
| --------------------------- | ------------------------------------------------------- |
| `bun dev`                   | Start Metro bundler                                     |
| `bun ios`                   | Run trên iOS simulator (cần app đã build)               |
| `npx expo run:ios`          | Build và run iOS app                                    |
| `bun android`               | Run trên Android emulator                               |
| `bun expo prebuild --clean` | Clean và generate lại thư mục native (`ios`, `android`) |
| Nhấn `r` trong Metro        | Reload app                                              |
| Cmd+R trong Simulator       | Reload app                                              |
| Cmd+D trong Simulator       | Mở Developer Menu                                       |

### Troubleshooting

**Lỗi: "No development build found"**

```bash
# Build lại native app
cd riz-app-v2
npx expo run:ios
```

**Lỗi: NitroModules compatibility**

```bash
# Đảm bảo react-native-nitro-modules đúng phiên bản
cd riz-app-v2
# Check package.json: "react-native-nitro-modules": "^0.32.0"
bun install
rm -rf ios node_modules
bun install
npx expo run:ios
```

**Clean rebuild**

Lựa chọn 1 (Thủ công):

```bash
cd riz-app-v2
rm -rf ios node_modules
bun install
npx expo run:ios
```

Lựa chọn 2 (Sử dụng Expo Prebuild - Khuyên dùng khi lỗi native):

```bash
cd riz-app-v2
bun expo prebuild --clean
```

**App không kết nối được Backend**

1. Đảm bảo Backend đang chạy: `curl http://localhost:4000/docs`
2. Kiểm tra `.env` có đúng `EXPO_PUBLIC_API_URL=http://localhost:4000`
3. Nếu chạy trên device thật, thay `localhost` bằng IP máy local

---

## Useful Commands

| Command                     | Mô tả                           |
| --------------------------- | ------------------------------- |
| **Backend**                 |                                 |
| `pnpm start:local:watch`    | Backend dev với hot reload      |
| `pnpm start:local:debug`    | Backend với debugger            |
| `pnpm test`                 | Chạy tests                      |
| **Frontend**                |                                 |
| `bun dev`                   | Frontend dev server             |
| `bun build`                 | Build Frontend production       |
| **Mobile App**              |                                 |
| `bun dev`                   | Start Metro bundler             |
| `bun ios`                   | Run iOS (app đã build)          |
| `npx expo run:ios`          | Build và run iOS                |
| `bun expo prebuild --clean` | Clean và rebuild native project |
| `bun android`               | Run Android                     |
| **Docker**                  |                                 |
| `docker-compose up -d`      | Start Docker containers         |
| `docker-compose down`       | Stop Docker containers          |
| `docker-compose logs -f`    | Xem Docker logs                 |

---

**Document created:** 20/01/2026
