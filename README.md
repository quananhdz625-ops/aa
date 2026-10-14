# aa
1. Giới thiệu
Bài thực hành sử dụng Apache Hadoop và Hadoop Streaming để phân tích doanh thu từ dữ liệu giao dịch thương mại điện tử.

Mục tiêu là tính tổng doanh thu theo từng danh mục sản phẩm (category), chỉ tính các giao dịch có trạng thái SUCCESS và bỏ qua các giao dịch FAILED hoặc RETURNED.

Chương trình sử dụng hai thành phần chính:

Mapper: Đọc dữ liệu đầu vào, lọc giao dịch thành công và phát ra cặp khóa – giá trị gồm danh mục sản phẩm và số tiền.
Reducer: Nhận các cặp dữ liệu đã được Hadoop nhóm theo khóa, sau đó cộng doanh thu của từng danh mục.
Kết quả được ghi ra file part-00000 và được sử dụng để xác định ba danh mục có doanh thu cao nhất.

2. Mục tiêu thực hành
Làm quen với HDFS và các thao tác quản lý dữ liệu trên HDFS.
Sử dụng Hadoop Streaming để chạy chương trình MapReduce viết bằng Python.
Thực hiện lọc dữ liệu giao dịch theo trạng thái.
Tính tổng doanh thu theo danh mục sản phẩm.
Kiểm tra kết quả đầu ra và phân tích các danh mục có doanh thu cao nhất.
3. Công nghệ sử dụng
Công nghệ	Vai trò
Apache Hadoop	Nền tảng xử lý dữ liệu lớn
HDFS	Lưu trữ dữ liệu đầu vào và kết quả
Hadoop Streaming	Cho phép sử dụng chương trình Python làm Mapper và Reducer
Python 3	Xử lý và tổng hợp dữ liệu
Google Colab	Môi trường thực hành
CSV	Định dạng dữ liệu đầu vào
4. Dữ liệu đầu vào
File dữ liệu: orders_2026_09.csv

Bộ dữ liệu chứa 1.500 giao dịch với 11 cột thông tin. Chương trình sử dụng hai trường chính:

status: Trạng thái giao dịch.
category: Danh mục sản phẩm.
amount: Giá trị giao dịch.
Phân loại trạng thái giao dịch:

Trạng thái	Số lượng
SUCCESS	1.240
FAILED	142
RETURNED	118
Tổng cộng	1.500
Chỉ những giao dịch có status = SUCCESS được sử dụng để tính doanh thu.

5. Quy trình xử lý dữ liệu
Quy trình thực hiện gồm các bước:

Khởi động các dịch vụ Hadoop cần thiết.
Tạo thư mục trên HDFS và tải file CSV lên.
Sử dụng Hadoop Streaming để chạy Mapper và Reducer.
Hadoop thực hiện giai đoạn Map, Shuffle/Sort và Reduce.
Đọc file kết quả part-00000 từ thư mục đầu ra trên HDFS.
Phân tích doanh thu theo danh mục và xác định Top 3.
Sơ đồ xử lý
orders_2026_09.csv
        |
        v
       HDFS
        |
        v
      Mapper
  Lọc SUCCESS và lấy
  category, amount
        |
        v
  Shuffle / Sort
 Nhóm theo category
        |
        v
      Reducer
 Cộng doanh thu từng nhóm
        |
        v
    part-00000
        |
        v
  Phân tích kết quả
      Top 3
Đường dẫn HDFS
Đường dẫn đầu vào:

/user/student/ecommerce/input/orders_2026_09.csv
Đường dẫn đầu ra của lần chạy sử dụng LocalJobRunner:

/user/student/ecommerce/output/revenue_by_category/
Log của lần chạy YARN ghi nhận thư mục đầu ra riêng:

/user/student/ecommerce/output/revenue_yarn/
Hai lần chạy được ghi lại riêng trong các file log. Cần kiểm tra kết quả ở từng thư mục tương ứng khi đối chiếu đầu ra.

6. Mã nguồn Mapper và Reducer
6.1. Mapper — mapper_revenue.py
Mapper đọc từng dòng CSV, kiểm tra trạng thái giao dịch và chỉ phát ra dữ liệu đối với giao dịch thành công.

Dữ liệu đầu ra của Mapper có dạng:

category amount
Ví dụ minh họa:

Electronics 1500000
Home 500000
Các giá trị trên chỉ minh họa định dạng đầu ra, không phải giao dịch được xác nhận từ bộ dữ liệu.

6.2. Reducer — reducer_revenue.py
Reducer nhận dữ liệu đã được Hadoop nhóm theo category, cộng các giá trị amount thuộc cùng một nhóm và xuất tổng doanh thu.

Đầu ra có dạng:

category total_revenue
Nhờ đó, kết quả cuối cùng chỉ còn một dòng cho mỗi danh mục sản phẩm.

7. Kết quả thực nghiệm
7.1. Tổng doanh thu theo danh mục
Kết quả được lưu trong part-00000 và result_revenue_by_category.txt.

Danh mục	Doanh thu (VNĐ)
Beauty	111.032.003
Book	39.306.022
Electronics	3.703.769.311
Fashion	218.597.763
Grocery	80.983.147
Home	410.315.065
Sport	166.194.354
Toy	83.881.779
Tổng cộng	4.814.079.444
7.2. Top 3 danh mục có doanh thu cao nhất
Hạng	Danh mục	Doanh thu (VNĐ)	Tỷ trọng doanh thu
1	Electronics	3.703.769.311	76,94%
2	Home	410.315.065	8,52%
3	Fashion	218.597.763	4,54%
File kết quả Top 3: top3.txt.

7.3. Nhận xét
Electronics có doanh thu cao nhất, chiếm khoảng 76,94% tổng doanh thu của các giao dịch thành công.
Home đứng thứ hai, với doanh thu 410.315.065 VNĐ.
Fashion đứng thứ ba, với doanh thu 218.597.763 VNĐ.
Ba danh mục đứng đầu đóng góp khoảng 90% tổng doanh thu thành công.
Kết quả cho thấy doanh thu tập trung đáng kể ở nhóm Electronics. Tuy nhiên, doanh thu cao không đồng nghĩa với lợi nhuận cao vì bài thực hành chưa tính giá vốn, chi phí vận chuyển và các chi phí kinh doanh khác.
8. Kiểm tra và đánh giá kết quả
Các file log được lưu trong repository để hỗ trợ kiểm tra từng bước thực hiện:

File	Nội dung
01_jps_dfsadmin_report.txt	Kiểm tra tiến trình Hadoop và trạng thái HDFS
02_hdfs_upload.txt	Ghi lại thao tác tải dữ liệu lên HDFS
03_fsck_block.txt	Kiểm tra file và block trên HDFS
04_streaming_run.txt	Log chạy Hadoop Streaming sử dụng LocalJobRunner
05_hdfs_output.txt	Kiểm tra dữ liệu đầu ra trên HDFS
06_yarn_run.txt	Log chạy Hadoop Streaming thông qua YARN
07_yarn_application_list.txt	Thông tin ứng dụng YARN
local_check.txt	File kiểm tra kết quả độc lập
Log LocalJobRunner ghi nhận 1.501 bản ghi đầu vào, bao gồm dòng tiêu đề CSV, và 1.240 bản ghi đầu ra từ Mapper. Log này cũng ghi nhận 8 nhóm dữ liệu đầu ra tương ứng với 8 danh mục sản phẩm.

Log YARN ghi nhận ứng dụng hoàn thành thành công. Tuy nhiên, các bộ đếm xử lý của hai lần chạy khác nhau; vì vậy, cần kiểm tra file đầu ra của lần chạy YARN trước khi kết luận hai lần chạy cho kết quả hoàn toàn giống nhau.

9. Hướng dẫn chạy lại
Bài thực hành được thực hiện trong môi trường Google Colab.

Mở notebook thực hành của bài.
Kết nối runtime.
Cài đặt và cấu hình Hadoop theo các bước trong notebook.
Khởi động các dịch vụ HDFS cần thiết.
Tải orders_2026_09.csv lên HDFS.
Chạy Hadoop Streaming với mapper_revenue.py và reducer_revenue.py.
Kiểm tra trạng thái hoàn thành của job.
Đọc file part-00000 và đối chiếu kết quả với các file đã lưu trong repository.
Một số lệnh kiểm tra HDFS:

hdfs dfs -ls /user/student/ecommerce/input

hdfs dfs -head /user/student/ecommerce/input/orders_2026_09.csv

hdfs dfs -ls /user/student/ecommerce/output/revenue_by_category

hdfs dfs -cat /user/student/ecommerce/output/revenue_by_category/part-00000
Lưu ý: Thư mục đầu ra phải chưa tồn tại trước khi chạy một job mới, hoặc cần được xử lý theo đúng quy trình của notebook.

10. Cấu trúc repository
DLL_01/
├── README.md
├── mapper_revenue.py
├── reducer_revenue.py
├── orders_2026_09.csv
├── part-00000
├── result_revenue_by_category.txt
├── top3.txt
├── top3_table.md
├── local_check.txt
├── 01_jps_dfsadmin_report.txt
├── 02_hdfs_upload.txt
├── 03_fsck_block.txt
├── 04_streaming_run.txt
├── 05_hdfs_output.txt
├── 06_yarn_run.txt
└── 07_yarn_application_list.txt
Sau khi bổ sung notebook, ảnh minh chứng và báo cáo, cập nhật cây thư mục này để phản ánh đúng các file thực tế.

11. Kết luận
Bài thực hành đã xây dựng quy trình xử lý dữ liệu giao dịch thương mại điện tử bằng Hadoop Streaming và Python. Mapper lọc giao dịch thành công, Reducer tổng hợp doanh thu theo danh mục, và kết quả được ghi ra HDFS.

Kết quả cho thấy Electronics là danh mục có doanh thu cao nhất trong bộ dữ liệu được phân tích. Các file mã nguồn, dữ liệu, kết quả và log được lưu trong repository để phục vụ việc kiểm tra và đối chiếu.
