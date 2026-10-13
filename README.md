# BÀI TẬP 1: XÂY DỰNG QUY TRÌNH CI ĐA PHIÊN BẢN JDK (MATRIX STRATEGY) CHO JAVA SPRING BOOT GRADLE

**Học phần:** Điện toán đám mây / CI-CD & DevOps (IT209)  
**Session:** 12 - Tối ưu hóa & Nâng cao CI/CD Pipeline với GitHub Actions  
**Thư mục bài làm:** `Lession12/Bai01`  
**GitHub Submission Path:** `homework/session_12/ex1/`  

---

## 1. Mục tiêu bài thực hành
- Thiết lập quy trình Tích hợp liên tục (CI) tự động kiểm thử đa môi trường sử dụng chiến lược ma trận (**Matrix Strategy**).
- Tự động hóa việc phân nhánh luồng kiểm thử thành 2 sub-job chạy song song trên 2 phiên bản JDK dài hạn (LTS) phổ biến nhất hiện nay: **Java JDK 17** và **Java JDK 21**.
- Tối ưu hóa thời gian thực thi ma trận bằng cách kích hoạt cơ chế bộ nhớ đệm phụ thuộc (**Gradle Dependency Caching**) với tham số `cache: 'gradle'`.
- Đảm bảo mã nguồn ứng dụng Spring Boot tương thích mượt mà trên cả hai môi trường runtime trước khi cho phép merge code vào nhánh `main`.

---

## 2. Cấu trúc thư mục dự án

```text
Bai01/
├── .github/
│   └── workflows/
│       └── ci-matrix.yml        # Kịch bản GitHub Actions Matrix CI đa phiên bản JDK
├── gradle/
│   └── wrapper/                 # Gradle Wrapper
├── src/
│   ├── main/java/               # Mã nguồn ứng dụng Spring Boot (Java 17/21 compatible)
│   └── test/java/               # Bộ Unit Test JUnit 5
├── build.gradle                 # Cấu hình Gradle build & toolchain tương thích
├── settings.gradle              # Tên định danh dự án
├── gradlew                      # Gradle wrapper script cho Linux/macOS
├── gradlew.bat                  # Gradle wrapper script cho Windows
└── README.md                    # Tài liệu hướng dẫn chi tiết & báo cáo kiểm thử
```

---

## 3. Nội dung tệp kịch bản `.github/workflows/ci-matrix.yml`

```yaml
name: Java Spring Boot Multi-JDK Matrix CI

on:
  push:
    branches: [ "main" ]

jobs:
  test-matrix:
    name: Test on Java ${{ matrix.java-version }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: ['17', '21']

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK ${{ matrix.java-version }}
        uses: actions/setup-java@v5
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền thực thi cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi Unit Test trên Java ${{ matrix.java-version }}
        run: ./gradlew test
```

---

## 4. Phân tích kỹ thuật Matrix Strategy trong GitHub Actions

### 4.1. Khối cấu hình ma trận `strategy.matrix`
```yaml
strategy:
  matrix:
    java-version: ['17', '21']
```
- **Cơ chế hoạt động:** Khi workflow được kích hoạt, GitHub Actions Engine sẽ đọc mảng cấu hình `['17', '21']` và tự động phân tách Job gốc `test-matrix` thành **2 phiên bản Job con (Sub-jobs)** hoàn toàn độc lập:
  1. `test-matrix (17)`: Chạy trên máy ảo `ubuntu-latest` với môi trường Java JDK 17 (Temurin).
  2. `test-matrix (21)`: Chạy trên máy ảo `ubuntu-latest` với môi trường Java JDK 21 (Temurin).
- **Thực thi song song (Parallel Execution):** Hai sub-jobs này được cấp phát tài nguyên runner riêng biệt và thực thi đồng thời, giúp tiết kiệm thời gian chờ đợi thay vì chạy tuần tự.

### 4.2. Truyền biến động `${{ matrix.java-version }}`
- **Đặt tên Job linh hoạt:**
  ```yaml
  name: Test on Java ${{ matrix.java-version }}
  ```
  Giúp giao diện trực quan hóa trên tab **Actions** hiển thị rõ ràng: `Test on Java 17` và `Test on Java 21`.
- **Cài đặt JDK tương ứng với `setup-java@v5`:**
  ```yaml
  - name: Thiết lập môi trường Java JDK ${{ matrix.java-version }}
    uses: actions/setup-java@v5
    with:
      java-version: ${{ matrix.java-version }}
      distribution: 'temurin'
      cache: 'gradle'
  ```
  Mỗi runner tự động tải và cấu hình đúng bản phân phối JDK được chỉ định mà không cần viết lặp code workflow.

### 4.3. Kết hợp với Gradle Dependency Caching
- Việc bật `cache: 'gradle'` trong ma trận giúp cả hai sub-jobs tận dụng bộ nhớ đệm của GitHub Actions.
- Key cache sẽ được tính toán riêng biệt phù hợp với phiên bản hệ điều hành và tệp build của từng môi trường, giúp các lần chạy tiếp theo của cả Java 17 và Java 21 đều đạt trạng thái **Cache Hit**, giảm tới 60-75% thời gian tải dependencies từ mạng.

---

## 5. Hướng dẫn kiểm tra và nộp bài

### Bước 1: Khởi tạo Git & Đẩy code lên GitHub
Mở terminal tại thư mục `d:/IT209/Lession12/Bai01`:
```bash
git init -b main
git add .
git commit -m "feat: setup multi-jdk matrix CI workflow for Java 17 and 21"
git remote add origin https://github.com/Vanh48420389/PTIT_CNTT3_IT209_Session12_Bai1.git
git push -u origin main
```

### Bước 2: Quan sát thực thi trên giao diện GitHub Actions
1. Mở repository trên GitHub, chuyển sang tab **Actions**.
2. Chọn lần chạy workflow mới nhất: **`Java Spring Boot Multi-JDK Matrix CI`**.
3. Quan sát cây tiến trình của Job:
   - Job tự động rẽ nhánh thành 2 sub-jobs:
     * `Test on Java 17`
     * `Test on Java 21`
   - Cả hai sub-jobs đều chạy độc lập, thực thi `./gradlew test` và chuyển sang trạng thái tích xanh (**Success**).

### Bước 3: Minh chứng nộp bài
- Chụp ảnh màn hình toàn bộ trang chi tiết của lần chạy hiển thị rõ:
  - Tên workflow `Java Spring Boot Multi-JDK Matrix CI`.
  - Cả 2 sub-job `Test on Java 17` và `Test on Java 21` đều đạt tích xanh (**Success**).
- Nộp tệp `.github/workflows/ci-matrix.yml` và ảnh chụp màn hình vào thư mục `homework/session_12/ex1/`.

---

## 6. Xác thực kiểm thử cục bộ (Local Verification)
Đã thực thi kiểm thử cục bộ bằng Gradle Wrapper:
```bash
./gradlew test
```
- **Kết quả:** `BUILD SUCCESSFUL in 25s` (5 actionable tasks: 5 executed).
- Toàn bộ bộ Unit Test (`StudentServiceTest`, `StudentControllerTest`, `StudentApplicationTests`) đều vượt qua 100%.
