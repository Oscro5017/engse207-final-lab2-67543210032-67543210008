# ENGSE207 Software Architecture
## README — Final Lab Set 1: Microservices + HTTPS + Lightweight Logging

---

## 1. ข้อมูลรายวิชาและสมาชิก

**รายวิชา:** ENGSE207 Software Architecture
**ชื่องาน:** Final Lab — ชุดที่ 1: Microservices + HTTPS + Lightweight Logging

**สมาชิกในกลุ่ม**
- นายวรรธนะ คำมาลัย รหัสนักศึกษา 67543210023-7
- นายณัฐพงศ์ จินะปัญญา รหัสนักศึกษา 67543210008-8

**Repository:** `final-lab-set1/`

---

## 2. ภาพรวมของระบบ

Final Lab ชุดที่ 1 เป็นการพัฒนาระบบ Task Board แบบ Microservices โดยเน้นหัวข้อสำคัญดังนี้

- การทำงานแบบแยก service
- การใช้ Nginx เป็น API Gateway
- การเปิดใช้งาน HTTPS ด้วย Self-Signed Certificate
- การยืนยันตัวตนด้วย JWT
- การจัดเก็บ log แบบ Lightweight Logging ผ่าน Log Service
- การเชื่อมต่อ Frontend กับ Backend ผ่าน HTTPS

งานชุดนี้ **ไม่มี Register** และใช้เฉพาะ **Seed Users** ที่กำหนดไว้ในฐานข้อมูล

---

## 3. วัตถุประสงค์ของงาน

งานนี้มีจุดมุ่งหมายเพื่อฝึกให้นักศึกษาสามารถ

- ออกแบบระบบแบบ Microservices ในระดับพื้นฐาน
- ใช้ Nginx เป็น reverse proxy และ TLS termination
- ใช้ JWT สำหรับ authentication ระหว่าง frontend และ backend
- ออกแบบ logging flow ผ่าน REST API และจัดเก็บ log ลงฐานข้อมูล
- ใช้ Docker Compose เพื่อรวมทุก service ให้ทำงานร่วมกันได้

---

## 4. Architecture Overview

```
Browser / Postman
       │
       │ HTTPS :443  (HTTP :80 → redirect HTTPS)
       ▼
┌──────────────────────────────────────────────────────┐
│  Nginx (API Gateway + TLS Termination + Rate Limiter) │
│                                                      │
│  /api/auth/*  → auth-service:3001  (ไม่ต้องมี JWT)   │
│  /api/tasks/* → task-service:3002  [JWT required]    │
│  /api/logs/*  → log-service:3003   [JWT required]    │
│  /            → frontend:80        (Static HTML)     │
└──────┬───────────────┬──────────────────┬────────────┘
       │               │                  │
       ▼               ▼                  ▼
┌────────────┐  ┌─────────────┐  ┌───────────────┐
│ Auth Svc   │  │ Task Svc    │  │ Log Service   │
│   :3001    │  │   :3002     │  │   :3003       │
│            │  │             │  │               │
│ • Login    │  │ • CRUD Tasks│  │ • POST /logs  │
│ • /verify  │  │ • JWT Guard │  │ • GET  /logs  │
│ • /me      │  │ • logEvent  │  │ • /stats      │
└─────┬──────┘  └──────┬──────┘  └───────────────┘
      │                │
      └───────┬─────────┘
              ▼
     ┌──────────────────┐
     │   PostgreSQL     │
     │  (shared DB)     │
     │  • users table   │
     │  • tasks table   │
     │  • logs table    │
     └──────────────────┘
```

### Services ที่ใช้ในระบบ

| Service | Port | หน้าที่ |
|---|---|---|
| nginx | 80, 443 | API Gateway, HTTPS, rate limiting |
| frontend | 80 | Task Board UI และ Log Dashboard |
| auth-service | 3001 | Login, Verify, Me |
| task-service | 3002 | CRUD Tasks + JWT middleware |
| log-service | 3003 | รับและแสดง logs |
| postgres | 5432 | shared database |

---

## 5. โครงสร้าง Repository

```
final-lab-set1/
├── README.md
├── TEAM_SPLIT.md
├── INDIVIDUAL_REPORT_67543210023-7.md
├── INDIVIDUAL_REPORT_67543210008-8.md
├── docker-compose.yml
├── .env.example
├── .gitignore
├── nginx/
│   ├── nginx.conf
│   ├── Dockerfile
│   └── certs/
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   └── logs.html
├── auth-service/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       ├── index.js
│       ├── routes/auth.js
│       ├── middleware/jwtUtils.js
│       └── db/db.js
├── task-service/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       ├── index.js
│       ├── routes/tasks.js
│       ├── middleware/
│       └── db/db.js
├── log-service/
│   ├── Dockerfile
│   ├── package.json
│   └── src/index.js
├── db/
│   └── init.sql
├── scripts/
│   └── gen-certs.sh
└── screenshots/
```

---

## 6. เทคโนโลยีที่ใช้

- Node.js / Express.js
- PostgreSQL
- Nginx
- Docker / Docker Compose
- HTML / CSS / JavaScript
- JWT (jsonwebtoken)
- bcryptjs

---

## 7. การตั้งค่าและการรันระบบ

### 7.1 สร้าง Self-Signed Certificate

```bash
chmod +x scripts/gen-certs.sh
./scripts/gen-certs.sh
```

คำสั่งนี้จะสร้างไฟล์ `nginx/certs/cert.pem` และ `nginx/certs/key.pem` อัตโนมัติ

### 7.2 สร้างไฟล์ `.env`

คัดลอกจาก `.env.example` แล้วกำหนดค่าตามต้องการ

```bash
cp .env.example .env
```

ตัวอย่างค่าใน `.env`

```env
POSTGRES_DB=taskboard
POSTGRES_USER=admin
POSTGRES_PASSWORD=secret123
JWT_SECRET=engse207-super-secret-change-me
JWT_EXPIRES=1h
```

### 7.3 สร้าง bcrypt hash สำหรับ Seed Users

กลุ่มของเราสร้าง bcrypt hash เองโดยใช้คำสั่งต่อไปนี้

```bash
node -e "const b=require('bcryptjs'); console.log(b.hashSync('alice123',10))"
node -e "const b=require('bcryptjs'); console.log(b.hashSync('bob456',10))"
node -e "const b=require('bcryptjs'); console.log(b.hashSync('adminpass',10))"
```

จากนั้นนำค่า hash ที่ได้ไปแทนค่า placeholder ใน `db/init.sql` ในส่วน `INSERT INTO users`

### 7.4 รันระบบ

```bash
docker compose down -v
docker compose up --build
```

รอจนทุก container ขึ้น healthy แล้วเปิด browser

### 7.5 เปิดใช้งานผ่าน Browser

- Frontend: `https://localhost`
- Log Dashboard: `https://localhost/logs.html`

> หมายเหตุ: เนื่องจากใช้ self-signed certificate browser จะขึ้นคำเตือนด้านความปลอดภัย ให้กดยอมรับเพื่อเข้าทดสอบ

---

## 8. อธิบายการทำงานของ HTTPS, JWT และ Logging

### HTTPS Flow

1. Browser ส่ง request มาที่ port 80
2. Nginx redirect ไปยัง port 443 ด้วย `301 Moved Permanently`
3. Nginx รับ HTTPS request และทำ TLS termination ด้วย self-signed certificate
4. Nginx forward request ต่อไปยัง service ที่เกี่ยวข้องผ่าน HTTP ภายใน Docker network

### JWT Flow

1. User login ด้วย email และ password ที่ `/api/auth/login`
2. Auth Service ตรวจสอบ password ด้วย bcrypt และออก JWT token
3. Frontend เก็บ token ไว้ใน localStorage
4. ทุก request ไปยัง Task Service และ Log Service ต้องแนบ `Authorization: Bearer <token>`
5. Service ตรวจสอบ token ด้วย `JWT_SECRET` ก่อนดำเนินการใดๆ

### Lightweight Logging Flow

1. เมื่อ auth-service หรือ task-service มี event สำคัญ จะเรียก `logEvent()` helper
2. `logEvent()` ส่ง POST request ไปยัง `http://log-service:3003/api/logs/internal` ภายใน Docker network
3. Log Service บันทึกลง PostgreSQL ใน table `logs`
4. Admin สามารถดู log ได้ผ่าน `GET /api/logs/` หรือหน้า `logs.html`

---

## 9. Seed Users สำหรับทดสอบ

| Username | Email | Password | Role |
|---|---|---|---|
| alice | alice@lab.local | alice123 | member |
| bob | bob@lab.local | bob456 | member |
| admin | admin@lab.local | adminpass | admin |

> ต้อง generate bcrypt hash จริงแล้วแทนค่าลงใน `db/init.sql` ก่อน login

---

## 10. API Summary

### Auth Service (:3001)

| Method | Path | Auth | คำอธิบาย |
|---|---|---|---|
| POST | /api/auth/login | ไม่ต้อง | Login รับ JWT token |
| GET | /api/auth/verify | ไม่ต้อง | ตรวจสอบ token |
| GET | /api/auth/me | JWT | ดูข้อมูล user ตัวเอง |
| GET | /api/auth/health | ไม่ต้อง | Health check |

### Task Service (:3002)

| Method | Path | Auth | คำอธิบาย |
|---|---|---|---|
| GET | /api/tasks/health | ไม่ต้อง | Health check |
| GET | /api/tasks/ | JWT | ดูรายการ tasks |
| POST | /api/tasks/ | JWT | สร้าง task ใหม่ |
| PUT | /api/tasks/:id | JWT | แก้ไข task |
| DELETE | /api/tasks/:id | JWT | ลบ task |

### Log Service (:3003)

| Method | Path | Auth | คำอธิบาย |
|---|---|---|---|
| POST | /api/logs/internal | ไม่ต้อง (internal) | รับ log จาก services |
| GET | /api/logs/ | JWT (admin) | ดูรายการ logs |
| GET | /api/logs/stats | JWT (admin) | ดูสถิติ logs |
| GET | /api/logs/health | ไม่ต้อง | Health check |

---

## 11. การทดสอบระบบ

### ลำดับการทดสอบ

1. รัน `docker compose up --build` และตรวจสอบว่าทุก container healthy
2. เปิด `https://localhost` และยืนยัน HTTP redirect ไป HTTPS
3. Login ด้วย seed users ทั้ง 3 บัญชี
4. ทดสอบ login ด้วย password ผิด → ต้องได้ 401
5. สร้าง task ใหม่ด้วย JWT
6. ดูรายการ tasks
7. แก้ไข task
8. ลบ task
9. ทดสอบเรียก API โดยไม่มี JWT → ต้องได้ 401
10. ทดสอบ Log Dashboard ด้วย member → ต้องได้ 403
11. ทดสอบ Log Dashboard ด้วย admin → ต้องเห็น logs
12. ทดสอบ rate limit โดยส่ง login ผิดหลายครั้ง → ต้องได้ 429

### ตัวอย่าง curl

```bash
BASE="https://localhost"

# Login และเก็บ token
TOKEN=$(curl -sk -X POST $BASE/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@lab.local","password":"alice123"}' | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['token'])")

# ดู tasks
curl -sk $BASE/api/tasks/ -H "Authorization: Bearer $TOKEN"

# สร้าง task
curl -sk -X POST $BASE/api/tasks/ \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Test task","priority":"high"}'

# ทดสอบไม่มี JWT
curl -sk $BASE/api/tasks/

# ดู logs ด้วย admin
ADMIN_TOKEN=$(curl -sk -X POST $BASE/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@lab.local","password":"adminpass"}' | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['token'])")
curl -sk $BASE/api/logs/ -H "Authorization: Bearer $ADMIN_TOKEN"
```

---

## 12. Screenshots ที่แนบในงาน

| ไฟล์ | รายการ |
|---|---|
| 01_docker_running.png | docker compose up ทุก container healthy |
| 02_https_browser.png | browser เข้า https://localhost ได้ |
| 03_login_success.png | login สำเร็จ ได้รับ JWT token |
| 04_login_fail.png | login ผิด password → 401 |
| 05_create_task.png | สร้าง task ใหม่ → 201 |
| 06_get_tasks.png | ดูรายการ tasks → 200 |
| 07_update_task.png | แก้ไข task → 200 |
| 08_delete_task.png | ลบ task → 200 |
| 09_no_jwt_401.png | เรียก API ไม่มี JWT → 401 |
| 10_logs_api.png | admin ดู logs สำเร็จ |
| 11_rate_limit.png | login เร็วเกินกำหนด → 429 |
| 12_frontend_screenshot.png | หน้า Task Board UI |

---

## 13. ปัญหาที่พบและแนวทางแก้ไข

**ปัญหาที่ 1: Nginx crash loop เพราะ resolve hostname ไม่ได้ตอน startup**
Nginx พยายาม resolve ชื่อ `auth-service`, `task-service`, `log-service` ตอนเริ่มต้น แต่ services ยังไม่พร้อม ทำให้ nginx exit และ restart วนซ้ำ
แก้โดยเปลี่ยน `depends_on` ใน docker-compose.yml ให้ใช้ `condition: service_started` แทนการระบุชื่อ service ตรงๆ และเพิ่ม `resolver 127.0.0.11` ใน nginx.conf เพื่อให้ resolve hostname แบบ dynamic

**ปัญหาที่ 2: auth-service และ task-service เชื่อมต่อ DB ผิด host**
`db.js` ของทั้งสอง service ใช้ค่า default host เป็น `auth-db` และ `task-db` ซึ่งไม่มีใน docker-compose แก้โดยเปลี่ยนเป็น `postgres` ให้ตรงกับชื่อ service ใน docker-compose.yml

**ปัญหาที่ 3: init.sql ซ้ำกันหลายที่และสร้าง schema ผิด**
มี `init.sql` อยู่ใน `auth-service/src/db/` และ `task-service/src/db/` ซึ่งสร้าง table ชื่อ `auth_users` และใช้ column `owner_id VARCHAR` ไม่ตรงกับโจทย์ แก้โดยลบไฟล์ซ้ำทิ้ง และใช้เฉพาะ `db/init.sql` ไฟล์เดียวที่ mount เข้า postgres container

**ปัญหาที่ 4: auth-service เรียก seedUsers() ด้วยข้อมูลผิด**
`seed.js` ใน auth-service seed email แบบ `alice@example.com` ไม่ใช่ `alice@lab.local` ตามโจทย์ และใช้ password เดียวกันหมด (`password123`) แก้โดยลบ `seed.js` และ `initDB()` ออกจาก index.js แล้วพึ่ง `db/init.sql` ที่ postgres รันตอน startup แทน

**ปัญหาที่ 5: node_modules และ .env ถูกนำขึ้น repository**
มีการ `git add .` ผิด directory ทำให้ `node_modules/`, `.env`, `nginx/certs/*.pem` ถูก stage แก้โดยตรวจสอบ `.gitignore` ให้ครอบคลุมไฟล์เหล่านี้ และใช้ `git rm --cached` เพื่อ unstage

**ปัญหาที่ 6: Docker volume เก่าทำให้ seed ใหม่ไม่ทำงาน**
หลังแก้ `init.sql` แล้ว login ยังไม่ได้เพราะ postgres ยังใช้ข้อมูลจาก volume เดิม แก้โดยรัน `docker compose down -v` ก่อนทุกครั้งที่แก้ schema

---

## 14. ข้อจำกัดของระบบ

- ใช้ self-signed certificate สำหรับการพัฒนา ไม่เหมาะสำหรับ production จริง
- ใช้ shared database เพียง 1 ก้อน ไม่ใช่ database-per-service
- ยังไม่มีระบบ register ใช้เฉพาะ seed users ที่กำหนดไว้
- Logging เป็นแบบ lightweight ไม่ใช่ centralized observability platform เต็มรูปแบบ
- เหมาะสำหรับการเรียนรู้ architecture ระดับพื้นฐานและการต่อยอดไป Set 2

---

## 15. การต่อยอดไปยัง Set 2

งาน Set 1 เป็นพื้นฐานสำคัญสำหรับ Set 2 โดยประเด็นที่จะต่อยอด ได้แก่

- เพิ่ม Register API และ User Service
- เปลี่ยนจาก shared DB ไปเป็น database-per-service
- Deploy บน Railway Cloud
- ออกแบบ gateway strategy สำหรับหลาย service

---

## 16. ภาคผนวก — ไฟล์สำคัญใน Repository

- `docker-compose.yml` — กำหนด services, networks, volumes ทั้งหมด
- `nginx/nginx.conf` — HTTPS config, reverse proxy, rate limiting
- `db/init.sql` — Database schema และ seed users
- `auth-service/src/routes/auth.js` — Login, verify, me endpoints
- `task-service/src/routes/tasks.js` — CRUD tasks endpoints
- `log-service/src/index.js` — Log receiver และ query endpoints
- `frontend/index.html` — Task Board UI
- `frontend/logs.html` — Log Dashboard

---

> เอกสารฉบับนี้เป็น README สำหรับงาน Final Lab Set 1 ของกลุ่ม จัดทำเพื่อประกอบการส่งงานในรายวิชา ENGSE207 Software Architecture
> มหาวิทยาลัยเทคโนโลยีราชมงคลล้านนา
