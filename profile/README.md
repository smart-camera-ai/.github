# Smart Camera AI Dashboard

Hệ thống quản lý Smart Camera AI với khả năng tìm kiếm bằng ngôn ngữ tự nhiên, thông báo thời gian thực, và phân tích thống kê.

## Flowchart Diagram

```mermaid
flowchart TB
    subgraph Clients["👥 Clients"]
        Browser["Web Browser"]
    end

    subgraph Services["🔧 Application Services"]
        Frontend["🎨 Frontend<br/>Next.js 14<br/>:3000"]
        Backend["⚙️ Backend API<br/>FastAPI<br/>:8000"]
        AI["🤖 AI Service<br/>YOLO + CLIP<br/>:8001"]
        Video["🎬 Video Service<br/>FFmpeg<br/>:8002"]
    end

    subgraph Storage["💾 Data Storage"]
        PostgreSQL[("PostgreSQL<br/>TimescaleDB<br/>:5432")]
        Redis[("Redis<br/>:6379")]
        Qdrant[("Qdrant<br/>Vector DB<br/>:6333")]
        MinIO[("MinIO<br/>Object Storage<br/>:9000")]
    end

    subgraph External["📷 External"]
        Cameras["IP Cameras<br/>RTSP"]
        Notify["📢 Notifications<br/>Telegram / SMS"]
    end

    %% Connections
    Browser <--> Frontend
    Frontend <--> Backend
    Backend <--> AI
    Backend <--> Video
    Backend --> PostgreSQL
    Backend --> Redis
    Backend --> Qdrant
    Backend --> Notify
    AI --> Qdrant
    Video --> MinIO
    Video <-- RTSP --> Cameras
    Video --> AI

    %% Styling
    classDef service fill:#4a90d9,stroke:#333,color:#fff
    classDef storage fill:#27ae60,stroke:#333,color:#fff
    classDef external fill:#8e44ad,stroke:#333,color:#fff

    class Frontend,Backend,AI,Video service
    class PostgreSQL,Redis,Qdrant,MinIO storage
    class Cameras,Notify external
```

## Tính Năng

### 🔍 Tìm Kiếm Thông Minh (Semantic Search)
- Tìm kiếm video/sự kiện bằng ngôn ngữ tự nhiên
- VD: "shipper tới giao hàng lúc mấy giờ?"
- Sử dụng CLIP embeddings + Qdrant vector database

### 📢 Thông Báo Tự Động
- Gửi cảnh báo qua nhiều kênh: Telegram, SMS, Call
- Tự động phát hiện: trộm cắp, té ngã, hành vi bất thường
- Cấu hình alert rules linh hoạt

### 📊 Phân Tích & Thống Kê
- Đếm số lượng người ra vào theo thời gian thực
- Thống kê số lượng khách tới mua hàng
- Heatmap hoạt động theo giờ/ngày
- Dashboard trực quan với biểu đồ

## Tech Stack

### Frontend
- **Next.js 14** - React framework với App Router
- **TypeScript** - Type safety
- **Tailwind CSS** - Styling
- **shadcn/ui** - UI components
- **TanStack Query** - Data fetching
- **Recharts** - Charts
- **HLS.js** - Video playback

### Backend
- **Python FastAPI** - API server
- **PostgreSQL** + **TimescaleDB** - Time-series data
- **Redis** - Caching & message queue
- **Qdrant** - Vector database cho semantic search
- **MinIO** - Object storage cho videos
- **Alembic** - Database migrations

### AI/ML
- **YOLOv8/v9** - Object detection
- **CLIP** - Vision-language model cho semantic search
- **DeepSORT** - Multi-object tracking
- **PyTorch** - Deep learning framework

### Video Processing
- **FFmpeg** - Video encoding (RTSP → HLS)
- **OpenCV** - Frame extraction
- **Python AV** - RTSP client

## Kiến Trúc Hệ Thống

```
┌─────────────────────────────────────────────────────────────┐
│                    Next.js Frontend                         │
│              (Dashboard, Search, Analytics)                 │
└─────────────────────────────────────────────────────────────┘
                         ↕ REST API / WebSocket
┌─────────────────────────────────────────────────────────────┐
│                   FastAPI Backend                           │
│        (Auth, Camera CRUD, Events, Notifications)          │
└─────────────────────────────────────────────────────────────┘
        ↕                    ↕                    ↕
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│ AI Service   │   │Video Service │   │  Databases   │
│ YOLO + CLIP  │   │ RTSP → HLS   │   │ PostgreSQL   │
│ Detection    │   │ FFmpeg       │   │ Redis        │
│ Embeddings   │   │ Streaming    │   │ Qdrant       │
└──────────────┘   └──────────────┘   │ MinIO        │
                                       └──────────────┘
        ↓                    ↓
┌──────────────────────────────────────────────────────────────┐
│           External Notification Services                     │
│     Telegram Bot API  •  Twilio SMS  •  Twilio Voice        │
└──────────────────────────────────────────────────────────────┘
```

## Yêu Cầu Hệ Thống

### Development
- **Docker** và **Docker Compose**
- **Python 3.11+**
- **Node.js 20+**
- **Git**

### Production (Khuyến nghị)
- **CPU**: 8+ cores
- **RAM**: 16GB+ (32GB nếu chạy AI models)
- **GPU**: NVIDIA GPU với CUDA support (để chạy YOLO và CLIP nhanh hơn)
- **Storage**: 500GB+ SSD (cho video recordings)

## Cài Đặt & Chạy

### 1. Clone Repository

```bash
git clone <repository-url>
cd smart-camera-ui
```

### 2. Cấu Hình Environment Variables

Copy các file `.env.example` thành `.env` và cập nhật giá trị:

```bash
# Backend
cp apps/backend/.env.example apps/backend/.env

# Frontend
cp apps/frontend/.env.example apps/frontend/.env

# AI Service
cp apps/ai-service/.env.example apps/ai-service/.env

# Video Service
cp apps/video-service/.env.example apps/video-service/.env
```

Cập nhật các giá trị sau trong `.env` files:
- `JWT_SECRET` - Secret key cho JWT authentication
- `TELEGRAM_BOT_TOKEN` - Token từ BotFather
- `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` - Credentials từ Twilio

### 3. Khởi Động Infrastructure (Databases)

Chỉ chạy PostgreSQL, Redis, Qdrant, MinIO:

```bash
cd infrastructure/docker
docker-compose up -d
```

Kiểm tra services đang chạy:

```bash
docker-compose ps
```

Services sẽ khởi động ở các ports:
- PostgreSQL: `localhost:5432`
- Redis: `localhost:6379`
- Qdrant: `localhost:6333` (API), `localhost:6334` (gRPC)
- MinIO: `localhost:9000` (API), `localhost:9001` (Console)

### 4. Khởi Động Backend API

```bash
cd apps/backend

# Cài đặt dependencies
pip install -r requirements.txt

# Chạy migrations (khi có)
# alembic upgrade head

# Khởi động server
uvicorn app.main:app --reload --port 8000
```

API docs sẽ có tại: http://localhost:8000/docs

### 5. Khởi Động Frontend

```bash
cd apps/frontend

# Cài đặt dependencies
npm install

# Chạy development server
npm run dev
```

Frontend sẽ chạy tại: http://localhost:3000

### 6. Khởi Động AI Service

```bash
cd apps/ai-service

# Cài đặt dependencies
pip install -r requirements.txt

# Download AI model weights (lần đầu tiên)
# Models sẽ tự động download khi khởi động

# Khởi động server
uvicorn app.main:app --reload --port 8001
```

AI Service API docs: http://localhost:8001/docs

### 7. Khởi Động Video Service

```bash
cd apps/video-service

# Cài đặt dependencies
pip install -r requirements.txt

# Khởi động server
uvicorn app.main:app --reload --port 8002
```

Video Service API docs: http://localhost:8002/docs

## Chạy Toàn Bộ Hệ Thống với Docker Compose

Để chạy tất cả services (bao gồm cả backend, frontend, AI, video):

```bash
cd infrastructure/docker
docker-compose --profile with-apps up -d
```

Services:
- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- AI Service: http://localhost:8001
- Video Service: http://localhost:8002
- MinIO Console: http://localhost:9001 (user: minioadmin, pass: minioadmin)

## Sử Dụng

### 1. Đăng Nhập

Tài khoản admin mặc định:
- **Email**: `admin@smartcamera.local`
- **Password**: `admin123`

### 2. Thêm Camera

1. Vào trang **Cameras**
2. Click **Add Camera**
3. Nhập thông tin:
   - Name: Tên camera
   - Location: Vị trí
   - RTSP URL: `rtsp://username:password@camera-ip:554/stream`
   - AI Features: Chọn `["object_detection", "person_counting"]`
4. Click **Save**

### 3. Cấu Hình Alert Rules

1. Vào trang **Alerts**
2. Click **Create Rule**
3. Cấu hình:
   - Name: "Phát hiện trộm cắp"
   - Event Type: `theft`
   - Conditions: `{"min_confidence": 0.8, "time_range": "18:00-06:00"}`
   - Channels: `["telegram", "sms"]`
   - Recipients: Telegram chat ID hoặc số điện thoại
4. Click **Save**

### 4. Tìm Kiếm Video

1. Vào trang **Search**
2. Nhập truy vấn bằng tiếng Việt:
   - "shipper tới giao hàng"
   - "người mặc áo đỏ"
   - "xe ô tô màu đen"
3. Kết quả sẽ hiển thị các video clip liên quan với similarity score

### 5. Xem Thống Kê

1. Vào trang **Analytics**
2. Xem các biểu đồ:
   - Overview: Tổng quan hôm nay
   - Person Counting: Số người ra vào theo thời gian
   - Events Timeline: Timeline các sự kiện
   - Heatmap: Hoạt động theo giờ/ngày

## Cấu Trúc Thư Mục

```
smart-camera-ui/
├── apps/
│   ├── backend/              # FastAPI backend
│   │   ├── app/
│   │   │   ├── api/v1/      # API endpoints
│   │   │   ├── models/      # Database models
│   │   │   ├── services/    # Business logic
│   │   │   ├── workers/     # Background tasks
│   │   │   └── main.py      # Entry point
│   │   ├── requirements.txt
│   │   └── Dockerfile
│   ├── frontend/             # Next.js frontend
│   │   ├── src/
│   │   │   ├── app/         # Next.js app router
│   │   │   ├── components/  # React components
│   │   │   ├── lib/         # Utilities
│   │   │   └── types/       # TypeScript types
│   │   ├── package.json
│   │   └── Dockerfile
│   ├── ai-service/           # AI models service
│   │   ├── app/
│   │   │   ├── models/      # YOLO, CLIP, etc.
│   │   │   ├── services/    # AI inference
│   │   │   └── main.py
│   │   ├── requirements.txt
│   │   └── Dockerfile
│   └── video-service/        # Video processing
│       ├── app/
│       │   ├── pipeline/    # RTSP, HLS encoding
│       │   └── main.py
│       ├── requirements.txt
│       └── Dockerfile
├── packages/
│   └── types/                # Shared TypeScript types
├── infrastructure/
│   └── docker/
│       ├── docker-compose.yml
│       └── init-db/          # Database init scripts
├── docs/                     # Documentation
├── scripts/                  # Utility scripts
└── README.md
```

## API Documentation

### Authentication

```bash
# Register
POST /api/v1/auth/register
{
  "email": "user@example.com",
  "password": "password123",
  "full_name": "User Name"
}

# Login
POST /api/v1/auth/login
{
  "username": "user@example.com",
  "password": "password123"
}
```

### Cameras

```bash
# List cameras
GET /api/v1/cameras

# Create camera
POST /api/v1/cameras
{
  "name": "Front Door",
  "location": "Entrance",
  "rtsp_url": "rtsp://camera:554/stream",
  "ai_features": ["object_detection", "person_counting"]
}

# Start camera streaming
POST /api/v1/cameras/{camera_id}/start
```

### Search

```bash
# Semantic search
POST /api/v1/search
{
  "query": "shipper tới giao hàng",
  "start_date": "2025-12-01T00:00:00Z",
  "end_date": "2025-12-10T23:59:59Z",
  "limit": 10
}
```

### Analytics

```bash
# Get overview stats
GET /api/v1/analytics/overview

# Get person count data
GET /api/v1/analytics/person-count?interval=1h&start_date=2025-12-01
```

Xem full API documentation tại: http://localhost:8000/docs

## Troubleshooting

### Database Connection Error

```bash
# Kiểm tra PostgreSQL đang chạy
docker ps | grep postgres

# Xem logs
cd infrastructure/docker
docker-compose logs postgres
```

### AI Models Not Loading

```bash
# Models sẽ tự động download lần đầu tiên
# Nếu lỗi, download thủ công:

cd apps/ai-service/models_weights

# YOLOv8
wget https://github.com/ultralytics/assets/releases/download/v0.0.0/yolov8n.pt

# CLIP model sẽ tự động download từ HuggingFace
```

### Video Streaming Issues

Đảm bảo FFmpeg đã được cài đặt:

```bash
# Ubuntu/Debian
sudo apt-get install ffmpeg

# macOS
brew install ffmpeg

# Windows
# Download từ: https://ffmpeg.org/download.html
```

### MinIO Access Denied

Kiểm tra MinIO buckets đã được tạo:

```bash
# Truy cập MinIO Console
# http://localhost:9001
# User: minioadmin, Pass: minioadmin

# Hoặc dùng mc CLI
docker exec -it smartcamera-minio-init /bin/sh
mc alias set myminio http://minio:9000 minioadmin minioadmin
mc ls myminio
```

## Development

### Running Tests

```bash
# Backend tests
cd apps/backend
pytest

# Frontend tests
cd apps/frontend
npm test
```

### Code Formatting

```bash
# Python (backend, AI, video services)
black .
isort .

# TypeScript (frontend)
npm run lint
npm run format
```

## Deployment

### Production Checklist

- [ ] Cập nhật `JWT_SECRET` với giá trị bảo mật
- [ ] Cấu hình HTTPS với reverse proxy (Nginx/Traefik)
- [ ] Setup proper PostgreSQL backups
- [ ] Configure MinIO for S3-compatible storage
- [ ] Setup monitoring (Prometheus + Grafana)
- [ ] Configure log aggregation
- [ ] Setup automated alerts
- [ ] Review and harden security settings

### Docker Production Deployment

```bash
cd infrastructure/docker
docker-compose -f docker-compose.prod.yml up -d
```

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## License

This project is licensed under the MIT License.

## Support

For issues and questions:
- GitHub Issues: [Create an issue]
- Email: support@smartcamera.local

---

Made with ❤️ by Smart Camera AI Team
