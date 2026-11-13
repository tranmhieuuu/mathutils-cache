# Bài 6 — CI/CD Pipeline Optimization & Caching

## Mục tiêu

Kế thừa Bài 5: `MathUtils`, `ReportPathResolver`, Logback, JUnit 5 và JaCoCo (line/branch >= 80%). Tối ưu GitHub Actions bằng cache Maven để tránh tải lại cùng dependency ở những lần chạy sau.

## Cách caching hoạt động

Workflow: `.github/workflows/ci.yml`, tự chạy cho `push` và `pull_request`, trên Ubuntu, Windows, macOS. Bước quan trọng:

```yaml
- name: Set up JDK 25 and Maven cache
  id: java
  uses: actions/setup-java@v6
  with:
    distribution: temurin
    java-version: '25'
    cache: 'maven'
    cache-dependency-path: 'pom.xml'
```

`setup-java` lưu và khôi phục Maven local repository (`~/.m2/repository`). Cache key phụ thuộc hệ điều hành, kiến trúc và đầu vào Maven (`pom.xml`). Nếu **đổi pom.xml**, cache có thể bị miss và tạo cache mới. Cache của ba hệ điều hành là riêng biệt.

Mục `Show Maven cache status` xuất trực tiếp:

- `MAVEN_CACHE_HIT=false`: chưa tìm thấy cache đúng key (ví dụ lần chạy đầu ở repository mới).
- `MAVEN_CACHE_HIT=true`: đã khôi phục cache đúng key, giúp Maven tái sử dụng dependency.
- `MAVEN_CACHE_KEY=...`: key xác định cache, so sánh key giữa hai lần chạy.

GitHub chỉ lưu cache vào **cuối job** khi có dữ liệu để lưu. Vì thế **phải đợi Push 1 hoàn tất** trước khi Push 2. Cache hit không đảm bảo hoàn toàn không truy cập Maven Central (plugin/dependency chưa có vẫn có thể tải), và không đảm bảo toàn bộ workflow luôn nhanh hơn.

## Hai lần push để lấy bằng chứng (khuyến nghị)

Dùng một repository GitHub **mới, trống** tên `mathutils-cache`, dùng `Bai06` làm gốc repository (trong GitHub phải thấy `.github/workflows/ci.yml` ngay từ gốc). Nếu chưa tạo repo, vào https://github.com/new.

Tại terminal IntelliJ / Git Bash trong `Bai06`:

```bash
git init
git branch -M main
git add .
git commit -m "Bai06: enable Maven cache"
git remote add origin https://github.com/USERNAME/mathutils-cache.git
git push -u origin main
```

Đổi `USERNAME` thành tài khoản thật, lấy URL bằng nút **Code → HTTPS** trên GitHub. Nếu đã có remote origin thì dùng `git remote set-url origin URL_DUNG`, không chạy `remote add` lần nữa.

**Push 1**: vào Actions, đợi cả ba OS chạy xong; mở job Ubuntu > `Show Maven cache status`. Ghi lại `MAVEN_CACHE_HIT` (thường `false` ở repo mới), `MAVEN_CACHE_KEY`, thời gian job Ubuntu ở trang job. Mở `Post Set up JDK 25 and Maven cache` (nếu thấy) để xem thông báo lưu cache. Chỉ ghi kết quả thực tế.

**Push 2**, sau khi Push 1 hoàn tất: sửa một dòng trong `README.md` (chẳng hạn thêm `Da kiem tra cache lan 2.` vào cuối README), lưu lại rồi:

```bash
git add README.md
git commit -m "docs: rerun CI to verify Maven cache hit"
git push
```

Quan trọng: **không sửa `pom.xml`**, không đổi branch, không đổi Java, không xóa cache giữa hai lần push. Mở Actions → lần chạy Push 2 → job Ubuntu > `Show Maven cache status`. Mục tiêu là `MAVEN_CACHE_HIT=true`. Ghi thời gian job Ubuntu của lần 2.

**Giải thích yêu cầu trước/sau:** Với đúng hai lần push, có thể so sánh **trước khi cache được làm nóng (cache miss)** và **sau khi cache đã được lưu (cache hit)**. Nếu bắt buộc so sánh một lần chạy hoàn toàn **không có** `cache: maven` với lần **có cache-hit**, cần thêm lần chạy thứ ba (không cache → warm cache → hit cache). Không được gọi một lần cache-miss là cache-hit.

## Xem logs và thời gian

Tại tab **Actions** → chọn run → job **Build and cache (ubuntu-latest)**:

1. Mở `Set up JDK 25 and Maven cache` để xem thông báo tìm/khôi phục cache.
2. Mở `Show Maven cache status` và đọc `MAVEN_CACHE_HIT` và `MAVEN_CACHE_KEY`.
3. Xem thời gian **job Ubuntu** hoặc cùng một bước `Maven clean verify` trong cả hai run; phải so sánh cùng loại thời gian.
4. Lưu ảnh vào `docs/`: `01_cache_miss.png`, `02_cache_hit.png`, `03_time_comparison.png` (nếu có).
5. Ghi số liệu thực tế vào `docs/CACHE_EVIDENCE.md`. Không điền số liệu giả.

## Chạy và nộp bài

Trong IntelliJ → Maven → **Execute Maven Goal**:

```text
clean verify
```

Kiểm thử + JaCoCo 80% phải thành công. Để copy JAR dependency dùng khi giáo viên chạy `run.sh`, thực hiện goal:

```text
dependency:copy-dependencies
```

Đảm bảo `lib/` có các `.jar`, rồi thử `bash run.sh` trong Git Bash. Trên Linux dùng `chmod +x run.sh && ./run.sh` (giảng viên sử dụng JDK 25, không cần Maven khi đã kèm lib/).

Nộp: `Bai06/` chứa `.github/`, `src/`, `lib/`, `docs/`, `pom.xml`, `run.sh`, `README.md`. **Không cần** `.git/`, `.idea/`, `target/`, `build/`. Nếu muốn giữ báo cáo JaCoCo HTML có thể copy `target/site/jacoco/` sang `docs/jacoco/` (không bắt buộc). Cache thực tế nằm trên GitHub runner/Actions, **không copy vào ZIP**.

## Kết luận

Dự án sử dụng cache Maven qua `setup-java`. Hiệu quả chỉ được xác nhận sau khi hai run thật cho thấy `cache miss` rồi `cache hit` và ghi số liệu thực tế từ GitHub Actions.
