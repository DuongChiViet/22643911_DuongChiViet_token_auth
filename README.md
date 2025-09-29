# Token Authentication Website

Một ứng dụng web Node.js sử dụng JWT (JSON Web Token) để xác thực người dùng với MongoDB và RESTful API.

## 📑 Mục lục

- [⚙️ Cài đặt & Chạy dự án](#️-cài-đặt--chạy-dự-án)
- [🔧 Set Up JWT Authentication](#-set-up-jwt-authentication)
- [🚫 Truy cập /profile khi chưa có token](#-truy-cập-profile-khi-chưa-có-token)
- [📝 Tạo tài khoản](#-tạo-tài-khoản)
- [🔑 Đăng nhập](#-đăng-nhập)
- [💾 Lưu trữ JWT Token](#-lưu-trữ-jwt-token)
- [🔐 Truy cập Protected Route](#-truy-cập-protected-route)
- [🚪 Đăng xuất](#-đăng-xuất)

## ⚙️ Cài đặt & Chạy dự án

### 📦 Cài đặt dependencies

```bash
npm install
```

### ▶️ Chạy dự án

```bash
node app.js
```

Ứng dụng sẽ chạy tại: `http://localhost:3000`

## 🔧 Set Up JWT Authentication

JWT (JSON Web Token) được sử dụng để xác thực người dùng thay vì session. Token được tạo khi đăng nhập thành công và gửi kèm trong mỗi request để xác thực:

```javascript
// Generate token trong login route
const token = jwt.sign({ id: user._id }, 'secretKey', { expiresIn: '1h' });

// Middleware để verify token
function authMiddleware(req, res, next) {
  const token = req.header('Authorization')?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'Access denied' });

  try {
    const verified = jwt.verify(token, 'secretKey');
    req.user = verified;
    next();
  } catch {
    res.status(400).json({ error: 'Invalid token' });
  }
}
```

📸 **Minh họa**: Token có thời hạn 1 giờ và được gửi trong header `Authorization: Bearer <token>`.

## 🚫 Truy cập /profile khi chưa có token

🎥 **Demo**: Khi người dùng cố gắng truy cập `/api/auth/profile` mà chưa có token:

**Request**:
```bash
GET http://localhost:3000/api/auth/profile
```

**Response**:
```json
{
  "error": "Access denied"
}
```

**Lý do**: Middleware `authMiddleware` kiểm tra token trong header `Authorization` trước khi cho phép truy cập.

## 📝 Tạo tài khoản

### Thông tin test:

```json
{
  "username": "duongchiviet",
  "email": "duongchiviet@example.com",
  "password": "123456"
}
```

### 📸 API Endpoint tạo tài khoản

**Request**:
```bash
POST http://localhost:3000/api/auth/register
Content-Type: application/json

{
  "username": "duongchiviet",
  "email": "duongchiviet@example.com",
  "password": "123456"
}
```

### ✅ Khi tạo tài khoản thành công

**Response**:
```json
{
  "message": "User registered successfully!"
}
```

### ❌ Khi tạo tài khoản thất bại

**Response** (nếu email đã tồn tại):
```json
{
  "error": "E11000 duplicate key error..."
}
```

### 🗄️ Kiểm tra database đã có thông tin chưa

Kiểm tra trong MongoDB:

```bash
mongo
use tokenAuthApp
db.users.find()
```

**Kết quả**:
```json
{
  "_id": "...",
  "username": "duongchiviet",
  "email": "duongchiviet@example.com",
  "password": "$2a$10$hashed_password_here"
}
```

## 🔑 Đăng nhập

### Thông tin test:

```json
{
  "email": "duongchiviet@example.com",
  "password": "123456"
}
```

### 📸 API Endpoint đăng nhập

**Request**:
```bash
POST http://localhost:3000/api/auth/login
Content-Type: application/json

{
  "email": "duongchiviet@example.com",
  "password": "123456"
}
```

### ✅ Khi đăng nhập thành công

**Response**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjY3MGY5ODc4ZjA1YjIzNDU2Nzg5MDEyMyIsImlhdCI6MTcyNzYxMjUzNiwiZXhwIjoxNzI3NjE2MTM2fQ.xyz..."
}
```

### ❌ Khi đăng nhập thất bại

**Response** (sai email):
```json
{
  "error": "User not found"
}
```

**Response** (sai mật khẩu):
```json
{
  "error": "Invalid credentials"
}
```

## 💾 Lưu trữ JWT Token

### 📸 Kết quả khi đăng nhập thành công

Token được trả về từ API và client cần lưu trữ để sử dụng cho các request tiếp theo:

**Cách lưu trữ token**:
1. **localStorage**: `localStorage.setItem('token', response.token)`
2. **sessionStorage**: `sessionStorage.setItem('token', response.token)`
3. **Cookie**: Có thể set cookie với token
4. **Memory**: Lưu trong biến JavaScript (mất khi refresh)

**Ví dụ sử dụng**:
```javascript
// Lưu token sau khi login
const response = await fetch('/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'test@example.com', password: '123456' })
});
const data = await response.json();
localStorage.setItem('token', data.token);
```

## 🔐 Truy cập Protected Route

### 📸 Truy cập /profile với token hợp lệ

**Request**:
```bash
GET http://localhost:3000/api/auth/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjY3MGY5ODc4ZjA1YjIzNDU2Nzg5MDEyMyIsImlhdCI6MTcyNzYxMjUzNiwiZXhwIjoxNzI3NjE2MTM2fQ.xyz...
```

**Response**:
```json
{
  "_id": "670f9878f05b23456789123",
  "username": "duongchiviet",
  "email": "duongchiviet@example.com"
}
```

### ❌ Truy cập với token không hợp lệ

**Response**:
```json
{
  "error": "Invalid token"
}
```

### ⏰ Truy cập với token đã hết hạn

**Response**:
```json
{
  "error": "Invalid token"
}
```

## 🚪 Đăng xuất

### Quy trình đăng xuất với JWT:

Với JWT, việc đăng xuất được thực hiện ở phía client bằng cách xóa token:

```javascript
// Xóa token từ localStorage
localStorage.removeItem('token');

// Hoặc xóa từ sessionStorage
sessionStorage.removeItem('token');

// Chuyển hướng về trang login
window.location.href = '/login';
```

### 👉 Sau khi đăng xuất:

✅ **Token bị xóa**: Token không còn được lưu trữ ở client

✅ **Truy cập bị từ chối**: Các request tiếp theo sẽ bị từ chối vì không có token

### 📸 Minh họa:

- **Trước khi logout**: Token được lưu trong localStorage/sessionStorage
- **Sau khi logout**: Token bị xóa, user không thể truy cập protected routes

## 🏗️ Cấu trúc dự án

```
📦 22643911_DuongChiViet_token_auth
├── 📄 app.js                 # File chính của ứng dụng
├── 📄 package.json           # Dependencies và scripts
├── 📄 README.md              # Tài liệu dự án
├── 📁 models/
│   └── 📄 User.js            # Model User cho MongoDB
└── 📁 routes/
    └── 📄 auth.js            # Routes xác thực với JWT
```

## 🔧 Dependencies

- **express**: Web framework
- **mongoose**: MongoDB object modeling
- **jsonwebtoken**: JWT token generation và verification
- **bcryptjs**: Password hashing
- **body-parser**: Parse request body

## 📡 API Endpoints

| Method | Endpoint             | Description              | Auth Required |
|--------|---------------------|--------------------------|---------------|
| POST   | `/api/auth/register` | Đăng ký tài khoản mới   | ❌            |
| POST   | `/api/auth/login`    | Đăng nhập                | ❌            |
| GET    | `/api/auth/profile`  | Lấy thông tin profile    | ✅            |

## 🔒 Security Features

- **Password Hashing**: Sử dụng bcrypt với salt rounds = 10
- **JWT Token**: Token có thời hạn 1 giờ
- **Protected Routes**: Middleware kiểm tra token cho các route cần authentication
- **Input Validation**: Kiểm tra email và password format
- **Error Handling**: Xử lý lỗi một cách an toàn

## 📝 Ghi chú

- MongoDB phải được khởi động trước khi chạy ứng dụng
- Default database: `tokenAuthApp`
- Default port: `3000`
- JWT secret key: `secretKey` (nên thay đổi trong production)
- Token expiry time: 1 giờ
- Password được mã hóa bằng bcrypt trước khi lưu vào database
- Sử dụng RESTful API thay vì traditional web forms