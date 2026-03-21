# Aurora Bank Strategic Customer Profiling & Revenue Optimization

## Introduction
Dựa trên bối cảnh chiến lược và các tập dữ liệu thực tế của Aurora Bank, đây là hướng dẫn lộ trình (roadmap) chi tiết để bạn triển khai một dự án Data Engineering (DE) hoàn chỉnh, từ hạ tầng đến trình diễn dữ liệu (Data Storytelling)

## Giai đoạn 1: Thiết kế Kiến trúc và Mô hình hóa Dữ liệu (Data Modeling)
Mục tiêu là chuyển đổi dữ liệu từ dạng thô (Raw) sang mô hình Star Schema để tối ưu hóa truy vấn cho Power BI/Tableau.

Thiết kế Star Schema:

Bảng Fact: Fact_Transactions (Lưu trữ các giao dịch, số tiền, mã lỗi, phương thức thanh toán).

Bảng Dimension:

Dim_Users: Thông tin khách hàng, điểm tín dụng, thu nhập. (Áp dụng SCD Type 2 để theo dõi biến động credit_score và yearly_income).

Dim_Cards: Loại thẻ, thương hiệu, hạn mức.

Dim_Merchants: Thông tin đơn vị chấp nhận thẻ, vị trí địa lý, mã MCC.

Dim_Date: Tạo bảng ngày chi tiết (Year, Quarter, Month, Day of Week, Is Weekend) để phân tích tính chu kỳ.

Lựa chọn Công nghệ (Tech Stack):

Ngôn ngữ: Python (Pandas/PySpark) để xử lý ETL.

Cơ sở dữ liệu: PostgreSQL (làm Data Warehouse cục bộ).

Điều phối (Orchestration): Apache Airflow để tự động hóa luồng chạy hàng ngày.

## Giai đoạn 2: Phát triển Đường ống ETL (Extract, Transform, Load)
Đây là giai đoạn cốt lõi để đảm bảo tính nhất quán (Consistency) và lũy đẳng (Idempotency) như chiến lược đã đề ra.

Extract (Trích xuất): Đọc dữ liệu từ các tệp CSV (users_data.csv, transactions_data.csv,...).

Transform (Biến đổi):

Làm sạch: Xử lý giá trị NULL, chuẩn hóa định dạng ngày tháng (date), loại bỏ các ký tự đặc biệt trong cột tiền tệ.

Tính toán chỉ số tài chính: Tính toán tỷ lệ DTI (Debt-to-Income) cho từng khách hàng tại thời điểm giao dịch.

Phân khúc (Segmentation): Áp dụng logic để gắn nhãn khách hàng (Ví dụ: credit_score > 800 => "Exceptional").

Xử lý lỗi: Phân tách các giao dịch có lỗi (Bad PIN, Insufficient Balance) vào một bảng riêng hoặc gắn flag để phân tích.

Load (Nạp):

Sử dụng phương pháp Upsert (Update if exists, Insert if not) dựa trên id để tránh trùng lặp dữ liệu khi chạy lại Pipeline.

## Giai đoạn 3: Phân tích Chuyên sâu và Logic Kinh doanh
Trước khi đưa lên Dashboard, bạn cần chuẩn bị các bảng "vàng" (Gold tables) đã được tính toán sẵn:

Bảng phân tích lỗi Merchant: Tổng hợp tỷ lệ lỗi theo từng merchant_id để xác định các đơn vị có thiết bị POS lỗi thời.

Bảng xu hướng Gen Z: Lọc khách hàng có năm sinh từ 1997-2012, tính toán các MCC (ngành hàng) yêu thích nhất của họ.

Bảng rủi ro tín dụng: Danh sách khách hàng có DTI > 43% và có dấu hiệu tăng nợ trong 3 tháng gần nhất.

## Giai đoạn 4: Xây dựng Dashboard (Power BI hoặc Tableau)
Dashboard không chỉ để xem số liệu, mà phải kể một câu chuyện về "Sức khỏe tài chính và Rủi ro".

1. Cấu trúc các trang báo cáo:
Trang 1: Executive Overview (Tổng quan điều hành)

KPI Cards: Tổng doanh thu (Interchange Fee), Tổng số giao dịch, Tỷ lệ giao dịch lỗi, Tỷ lệ nợ xấu (NPL).

Biểu đồ đường (Trend): Khối lượng giao dịch theo thời gian (Online vs. Chip vs. Swipe).

Biểu đồ bản đồ (Heatmap): Phân bổ chi tiêu theo địa lý (California, Florida,...).

Trang 2: Customer Insights (Chân dung khách hàng)

Biểu đồ cột chồng: Phân khúc khách hàng theo điểm tín dụng (FICO Buckets).

Scatter Chart: Mối tương quan giữa Thu nhập và Tổng nợ (để thấy nhóm DTI cao).

Slicer: Lọc theo thế hệ (Gen Z, Millennials) để xem thói quen tiêu dùng khác biệt.

Trang 3: Operational Risk & Fraud (Rủi ro và Gian lận)

Table Visual: Top 10 Merchants có tỷ lệ lỗi "Bad PIN" cao nhất.

Gauge Chart: Tỷ lệ giao dịch nghi ngờ gian lận (dựa trên các giao dịch không dùng chip hoặc giao dịch ở xa vị trí thường trú).

2. Kỹ thuật Visualization cần áp dụng:
Dax (Power BI) hoặc Calculated Fields (Tableau): Tạo các measure như YoY Growth % (Tăng trưởng so với năm trước).

Color Theme: Sử dụng màu sắc nhất quán (Xanh: Tăng trưởng/An toàn; Đỏ: Lỗi/Rủi ro).

Drill-through: Cho phép người dùng click vào một phân khúc khách hàng để xem chi tiết các giao dịch của nhóm đó.

## Giai đoạn 5: Kiểm soát Chất lượng và Vận hành (Data Quality Control)
Để đảm bảo hệ thống đạt mức "Middle-level", bạn cần bổ sung:

Data Quality Checks (Great Expectations): Tự động kiểm tra nếu amount bị âm hoặc card_number không đúng 16 chữ số.

Monitoring: Thiết lập thông báo qua Slack/Email nếu Pipeline thất bại hoặc dữ liệu không được cập nhật quá 2 giờ (Data Freshness).

Documentation: Viết tài liệu Data Dictionary (từ các tệp dictionary đính kèm) để người dùng Dashboard hiểu rõ ý nghĩa từng trường dữ liệu.

Checklist hoàn thành dự án:
[ ] Dữ liệu có bị trùng lặp khi chạy lại Pipeline không? (Tính lũy đẳng).

[ ] Các chỉ số DTI có khớp với công thức tài chính không?

[ ] Dashboard có trả lời được câu hỏi: "Nhóm khách hàng nào nên được mời nâng cấp thẻ tín dụng?" không?

[ ] Hệ thống có cảnh báo được các Merchant có thiết bị lỗi không?

DTI & Risk Scoring (Gold Layer): Chúng ta không chỉ lưu dữ liệu mà đã tính toán trực tiếp cột dti_ratio và gán nhãn risk_segment. Khi Power BI kết nối vào bảng này, nó chỉ việc hiển thị biểu đồ mà không cần tính toán lại (giảm tải cho BI tool).

Gen Z Profiling: Bằng cách Join bảng dim_users và fact_transactions ngay tại Gold layer, chúng ta tạo ra một "View" đặc thù cho các chiến dịch Marketing, giúp lọc nhanh các ngành hàng (MCC) mà giới trẻ đang chi tiêu nhiều nhất.

Merchant Health: Bảng merchant_error_monitoring cho phép quản trị viên ngân hàng nhìn thấy ngay Merchant ID nào đang có tỷ lệ "Bad PIN" cao bất thường để can thiệp kỹ thuật.

Hướng dẫn vận hành trên Databricks:
Workflow: Sử dụng Databricks Workflows để kết nối 3 Notebook này lại thành một Pipeline chạy hàng ngày.

Optimization: Sử dụng lệnh OPTIMIZE gold.customer_risk_analysis ZORDER BY (user_id) để tăng tốc độ truy vấn cho các báo cáo cá nhân hóa.

Governance: Sử dụng Unity Catalog để phân quyền: Data Engineers có quyền truy cập Bronze/Silver, trong khi Data Analysts chỉ cần quyền đọc ở tầng Gold.
