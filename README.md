# Todoplan — Full-Stack Todo List Application

แอปพลิเคชันบริหารจัดการงาน (Todo List) สไตล์โมเดิร์น Dark Forest ทำงานแบบ Full-Stack เต็มรูปแบบ พร้อมระบบทดสอบอัตโนมัติ (Automated API Tests) และ Docker Compose

---

## 🛠 Tech Stack

### Frontend
- **Framework**: [Vue 3](https://vuejs.org/) (`<script setup>`, Composition API)
- **Language**: [TypeScript](https://www.typescriptlang.org/) (Strict type checking ด้วย `vue-tsc`)
- **Styling**: [Tailwind CSS 4](https://tailwindcss.com/) (Dark Forest Theme `#0c1510` / Glassmorphism)
- **Build Tool**: [Vite 8](https://vite.dev/)
- **UX/UI**: Mobile-first Responsive Design, Fast Optimistic Updates, Double-click Inline Edit, Swipeable Kanban Board, Thumb-friendly Action Buttons

### Backend
- **Framework**: [AdonisJS 5](https://adonisjs.com/) (Node.js TypeScript Framework)
- **ORM**: Lucid ORM + PostgreSQL 16
- **Database**: PostgreSQL (พร้อม Indexing คอลัมน์ `completed` เพื่อประสิทธิภาพสูงสุด)
- **Validation**: AdonisJS Built-in Validator (ตัด whitespace อัตโนมัติ, ป้องกัน Empty Title, จำกัดความยาว 255 ตัวอักษร)
- **Error Handling**: Centralized Standard JSON Exception Handler (`code`, `message`, `errors`)
- **API Specification**: RESTful API พร้อม Health Check endpoint (`/health`)

### DevOps & Testing
- **Containerization**: Multi-container Docker Compose (`PostgreSQL` + `AdonisJS API` + `Vue 3 Nginx`)
- **Automated Testing**: Postman Collection (17 Requests, 73 Assertions ผ่าน 100% รันด้วย Newman CLI)

---

## 📁 โครงสร้างโปรเจกต์ (Project Structure)

```text
Todo/
├── docker-compose.yml                      # รันทั้งระบบ (DB + Backend + Frontend) ด้วยคำสั่งเดียว
├── README.md                               # เอกสารภาพรวมและคู่มือเริ่มต้น
├── .gitignore                              # ละเว้น .env, node_modules, build artifacts
│
├── todo-backend/todo-backend/              # AdonisJS 5 RESTful API Server
│   ├── app/
│   │   ├── Controllers/Http/TodosController.ts   # CRUD logic
│   │   ├── Exceptions/Handler.ts                 # Centralized error handler
│   │   ├── Models/Todo.ts                        # Lucid ORM Model
│   │   └── Validators/                           # Create/Update validators
│   ├── database/migrations/                      # ตาราง PostgreSQL พร้อม Index
│   ├── postman/
│   │   └── Todo-API.postman_collection.json      # ชุดทดสอบ API 17 Requests 73 Assertions
│   ├── Dockerfile
│   └── README.md
│
└── todo-frontend/todo-frontend/            # Vue 3 Frontend Web Application
    ├── src/
    │   ├── api/client.ts                   # Fetch API wrapper
    │   ├── components/                     # TodoItem, TodoForm, KanbanBoard, etc.
    │   ├── composables/                    # useTodos, useTheme
    │   ├── types/todo.ts                   # TypeScript interfaces
    │   └── App.vue                         # Main Application View
    ├── Dockerfile                          # Multi-stage build + Nginx static serve
    ├── nginx.conf
    └── README.md
```

---

## 🚀 เริ่มต้นใช้งานอย่างรวดเร็ว (Quick Start)

### วิธีที่ 1: รันด้วย Docker Compose (แนะนำ สะดวกที่สุด)

รันคำสั่งเดียวเพื่อเปิดระบบทั้งหมด (ฐานข้อมูล + Backend API + Frontend Web):

```bash
docker compose up -d --build
```

- **Frontend Web**: [http://localhost:8080](http://localhost:8080)
- **Backend API**: [http://localhost:3333](http://localhost:3333)
- **API Health Check**: [http://localhost:3333/health](http://localhost:3333/health)
- **PostgreSQL**: `localhost:5433` (User/Password: `postgres`)

ปิดระบบ:
```bash
docker compose down
```

---

### วิธีที่ 2: รันแบบแยกเครื่องในโหมด Development

#### 1. เปิด PostgreSQL
```bash
docker compose up -d db
```

#### 2. เริ่มต้น Backend API
```bash
cd todo-backend/todo-backend
npm install
cp .env.example .env
node ace generate:key
node ace migration:run
npm run dev
```
เซิร์ฟเวอร์จะเปิดที่ `http://localhost:3333`

#### 3. เริ่มต้น Frontend Web
```bash
cd todo-frontend/todo-frontend
npm install
npm run dev
```
เว็บจะเปิดที่ `http://localhost:5173`

---

## 📡 สรุป RESTful API Endpoints

| Method | Endpoint | คำอธิบาย | Status Code สำเร็จ |
| :--- | :--- | :--- | :---: |
| `GET` | `/health` | Health Check (Database & Node Env) | `200 OK` |
| `GET` | `/api/todos` | ดึงรายการ Todos ทั้งหมด (รองรับ `?completed=true/false`) | `200 OK` |
| `POST` | `/api/todos` | สร้างรายการ Todo ใหม่ (รับ `title`) | `201 Created` |
| `GET` | `/api/todos/:id` | ดึงรายการ Todo ตาม ID | `200 OK` |
| `PUT` | `/api/todos/:id` | แก้ไขชื่อ Todo | `200 OK` |
| `PATCH` | `/api/todos/:id` | ปรับสถานะเสร็จสิ้น (`completed`) หรือชื่อ | `200 OK` |
| `DELETE` | `/api/todos/:id` | ลบรายการ Todo | `204 No Content` |

---

## 🧪 การทดสอบอัตโนมัติด้วย Postman / Newman

คอลเลกชันการทดสอบอยู่ที่ `todo-backend/todo-backend/postman/Todo-API.postman_collection.json` ครอบคลุม:
- การทดสอบ CRUD ปกติ
- การทดสอบ Error Cases (Empty title, Missing title, Invalid boolean, 404 Not Found)
- Response Time ทุก Request ต้องไม่เกิน 3000ms

รันชุดทดสอบอัตโนมัติผ่าน Command Line ด้วย Newman:
```bash
npx newman run todo-backend/todo-backend/postman/Todo-API.postman_collection.json --env-var "baseUrl=http://127.0.0.1:3333"
```
**ผลลัพธ์: 17 Requests, 73 Assertions ผ่าน 100% (0 Failures)**

---

## 📱 Mobile-First Design & Safari Compatibility

1. **Touch Ergonomics**: ปุ่มทุกปุ่มมีขนาดพื้นที่แตะขั้นต่ำตามมาตรฐานมือถือ
2. **iOS Safari Auto-zoom Prevention**: Input text fields ใช้ขนาด `16px` ป้องกัน Safari ซูมหน้าจอขณะพิมพ์
3. **Safe Area Inset**: ออกแบบรองรับแถบ Home Indicator ด้านล่างของ iPhone (`env(safe-area-inset-bottom)`)
4. **Vite Proxy**: เชื่อมต่อผ่าน relative `/api` เพื่อตัดปัญหา CORS และ Mixed Content ในทุก Network
