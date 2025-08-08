# Printik Logging Stack

Hệ thống logging và monitoring cho dự án Printik, sử dụng Grafana Loki để thu thập và quản lý logs từ ứng dụng backend.

## Tổng quan

Stack này bao gồm:
- **Loki**: Hệ thống thu thập và lưu trữ logs
- **Promtail**: Agent thu thập logs từ ứng dụng
- **Grafana**: Giao diện visualization và query logs

## Cấu trúc thư mục

```
printik-logs/
├── docker-compose.yml           # Cấu hình Docker Compose chính
├── loki-config.yaml             # Cấu hình Loki server
├── promtail-config.yaml         # Cấu hình Promtail agent
├── grafana/
│   └── provisioning/
│       └── datasources/
│           └── loki.yaml        # Data source Loki cho Grafana
└── README.md                    # Tài liệu này
```

## Cách sử dụng

### 1. Khởi động hệ thống

```bash
docker-compose up -d
```

### 2. Truy cập các service

- **Grafana**: http://localhost:3001
  - Username: `admin`
  - Password: `admin123`
- **Loki**: http://localhost:3100
- **Promtail**: http://localhost:9080

### 3. Dừng hệ thống

```bash
docker-compose down
```

Để xóa cả volumes (dữ liệu):
```bash
docker-compose down -v
```

## Cấu hình

### Loki Configuration (`loki-config.yaml`)

- **Port**: 3100
- **Storage**: Filesystem với retention 744h (31 ngày)
- **Authentication**: Disabled (development mode)
- **Schema**: v11 với boltdb-shipper

### Promtail Configuration (`promtail-config.yaml`)

- **Log Path**: `/app/logs/development/*.log`
- **Job Name**: `printik-be`
- **Labels**: `level`, `service`, `environment`
- **JSON Parsing**: Tự động parse JSON logs với các fields:
  - `timestamp`
  - `level`
  - `message`
  - `service`
  - `environment`

### Grafana Configuration

- **Port**: 3001 (mapped từ 3000)
- **Admin Credentials**: admin/admin123
- **Data Source**: Loki tự động được cấu hình
- **Sign-up**: Disabled

## Log Format

Ứng dụng backend cần tạo logs theo format JSON:

```json
{
  "timestamp": "2024-01-01T12:00:00.000Z",
  "level": "info",
  "message": "Application started",
  "service": "printik-backend",
  "environment": "development"
}
```

## Troubleshooting

### Kiểm tra trạng thái containers

```bash
docker-compose ps
```

### Xem logs của containers

```bash
# Xem logs của tất cả services
docker-compose logs

# Xem logs của service cụ thể
docker-compose logs loki
docker-compose logs promtail
docker-compose logs grafana
```

### Restart service

```bash
docker-compose restart [service-name]
```

### Kiểm tra volumes

```bash
# Xem danh sách volumes
docker volume ls

# Xem chi tiết volume
docker volume inspect printik-logs_loki_data
docker volume inspect printik-logs_grafana_data
```

## Monitoring và Alerts

### Log Queries trong Grafana

Sau khi đăng nhập Grafana, bạn có thể sử dụng LogQL để query logs:

```logql
# Tất cả logs
{job="printik-be"}

# Logs theo level
{job="printik-be", level="error"}

# Logs theo service
{job="printik-be", service="printik-backend"}

# Logs trong khoảng thời gian
{job="printik-be"} |= "error" | json | timestamp > "2024-01-01T00:00:00Z"
```

### Tạo Dashboard

1. Đăng nhập Grafana
2. Tạo dashboard mới
3. Thêm panel với query LogQL
4. Cấu hình visualization (logs, table, etc.)

## Development

### Thêm log sources mới

Để thu thập logs từ nguồn khác, cập nhật `promtail-config.yaml`:

```yaml
scrape_configs:
  - job_name: new_logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: new-service
          __path__: /path/to/logs/*.log
```

### Cấu hình retention

Để thay đổi thời gian lưu trữ logs, cập nhật `loki-config.yaml`:

```yaml
limits_config:
  retention_period: 168h  # 7 ngày
```

## Production Considerations

Trước khi deploy production:

1. **Bật authentication** cho Loki
2. **Cấu hình persistent storage** (S3, GCS, etc.)
3. **Thiết lập monitoring** cho Loki và Promtail
4. **Cấu hình backup** cho dữ liệu logs
5. **Thiết lập alerts** cho log errors
6. **Cấu hình log rotation** và retention policies
