# Báo cáo Bài 6 — Maven Dependency Caching

Repository GitHub: (dán URL thực tế)

## Cấu hình

- `actions/setup-java@v6` với `cache: 'maven'` và `cache-dependency-path: 'pom.xml'`.
- Kiểm thử trên Ubuntu / Windows / macOS.
- Maven `clean verify` chạy JUnit, đóng gói JAR và kiểm tra JaCoCo >= 80%.

## Kết quả hai lần push

| Chỉ tiêu (cùng job ubuntu-latest) | Push 1 | Push 2 |
|---|---|---|
| Link Actions run | (điền) | (điền) |
| Thời gian job Ubuntu | (điền) | (điền) |
| `MAVEN_CACHE_HIT` | (điền theo log) | (điền theo log) |
| `MAVEN_CACHE_KEY` | (điền) | (điền) |
| Kết quả `Maven clean verify` | (điền) | (điền) |

## Minh chứng logs

- Ảnh Push 1: `01_cache_miss.png` (nếu đã chụp)
- Ảnh Push 2: `02_cache_hit.png` (nếu đã chụp)
- Ảnh so sánh thời gian: `03_time_comparison.png` (nếu đã chụp)

## Phân tích

Push đầu: (ghi lại thông báo cache miss / save cache *thực tế*).

Push thứ hai: (ghi lại thông báo cache restored / hit *thực tế*).

Nhận xét về thay đổi thời gian: (điền sau khi có số liệu thực; dao động runner là bình thường).

**Lưu ý:** Mục tiêu hai lần push là so sánh cache miss với cache hit. Nếu phải so sánh một bản không bật cache và một bản đã hit, cần tối thiểu ba run để làm nóng cache.
