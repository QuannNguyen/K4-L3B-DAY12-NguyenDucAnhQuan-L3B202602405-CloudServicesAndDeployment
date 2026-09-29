# Thông Tin Deploy — Checkpoint 5

> File này đã được cập nhật cho phương án dự phòng local với Docker Compose.
> CP5 sẽ kiểm tra `http://localhost:8000` khi `LOCAL_FALLBACK=true` trong `.env`.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Đức Anh Quân |
| Mã học viên | L3B202602405 |
| Repo | https://github.com/your-user/K4-L3B-DAY12-NguyenDucAnhQuan-L3B202602405-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | http://localhost:8000 |
| Platform | Railway (local fallback via Docker Compose) |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và nguồn giá trị, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | cổng local, hoặc platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong `.env` / dashboard, không lưu trong repo |
| `REDIS_URL` | ✅ | Redis service trong docker compose |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i http://localhost:8000/health

# 2. Readiness — mong đợi 200 {"status":"ready"}
curl -i http://localhost:8000/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: cp5-test" \
  -d '{"question":"Deploy là gì?"}'
```

## Kết Quả Chạy Thật

```text
health -> 200 ok
ready -> 200 ready
ask without key -> 401
ask with key -> 200 response
```

## Ảnh Chụp Màn Hình

Ảnh đã đặt trong `screenshots/` để minh chứng stack đang chạy local.

---

## Ghi Chú Phương Án Dự Phòng

Không deploy lên cloud, nên dùng local fallback bằng Docker Compose. Tất cả service và Redis đang chạy trên máy local với `LOCAL_FALLBACK=true` trong `.env`.
