# Session 12 – Bài 5: Chẩn đoán và khắc phục 3 sự cố Pipeline CI/CD

Tệp kịch bản: `.github/workflows/broken-pipeline.yml`
Quy trình: sửa từng lỗi, mỗi lần sửa là một commit, kiểm tra log trên tab Actions sau mỗi lần push.

## Lỗi 1 – Permission Denied
- **Bước lỗi:** Thực thi JUnit Test với Gradle
- **Log:** `./gradlew: Permission denied` – `Process completed with exit code 126`
- **Nguyên nhân:** File `gradlew` được commit với mode `644`, thiếu quyền thực thi (x). Runner Linux checkout đúng mode này nên không chạy được script.
- **Khắc phục:** Thêm bước trước lệnh test:
  - `run: chmod +x gradlew`

## Lỗi 2 – Authentication Failed
- **Bước lỗi:** Đăng nhập GitHub Container Registry (`docker/login-action@v3`)
- **Log:** `Error: Password required`
- **Nguyên nhân:** Gõ sai tên secret `secrets.REGISTRY_TOKN`. Secret không tồn tại thì GitHub trả về chuỗi rỗng, nên password truyền vào bị trống và đăng nhập thất bại.
- **Khắc phục:** Sửa thành `password: ${{ secrets.REGISTRY_TOKEN }}`, đúng tên secret đã tạo trong *Settings → Secrets and variables → Actions*.

## Lỗi 3 – Build Context Error
- **Bước lỗi:** Build và Push Docker Image (`docker/build-push-action@v5`)
- **Log:** `unable to prepare context: path "./docker" not found`
- **Nguyên nhân:** Khai báo `context: ./docker` và `file: ./docker/Dockerfile`, nhưng repo không có thư mục `docker/`. Dockerfile nằm ở thư mục gốc.
- **Khắc phục:** Đổi thành `context: .` và `file: ./Dockerfile`.

## Kết quả
- Sau 3 commit sửa lỗi, pipeline chuyển sang trạng thái tích xanh.
- Image được đẩy lên `ghcr.io/ngthekien/it209_ss12_ex05:latest`.

## Bài học rút ra
- Luôn đọc log từ **bước đầu tiên bị đỏ**. Các bước sau đó bị bỏ qua (skipped) nên không chạy tới.
- Tên secret phân biệt chữ hoa, chữ thường và GitHub không báo lỗi khi gõ sai tên, chỉ trả về giá trị rỗng.
- `context` là thư mục gửi cho Docker để build. Đường dẫn `file` và các lệnh `COPY` đều tính từ thư mục này.
