##### 2023 COFFEE CHAIN REVENUE ANALYSIS REPORT



###### **1.Tổng quan**

  Dự án này tập trung vào việc phân tích doanh thu chuỗi cà phê 6 tháng đầu năm nhằm cung cấp cái nhìn toàn diện về doanh thu. Với mục tiêu là giúp người quản lý đưa ra quyết định dựa trên dữ liệu.



###### **2.Công cụ và Kỹ thuật sử dụng**

  Power BI: Thiết kế Dashboard, báo cáo tương tác và trực quan hóa dữ liệu.

&nbsp; DAX: Xây dựng các Measure để tính toán

  Excel: Nguồn dữ liệu đầu vào.

  Power Query: Chuẩn hóa dữ liệu, thay đổi kiểu dữ liệu và tạo sơ đồ quan hệ.



###### **3.Bối cảnh dự án và vấn đề kinh doanh**

&nbsp; Bối cảnh dự án: Công ty là một chuỗi cà phê với hơn 3 chi nhánh trên toàn quốc. Trong 6 tháng qua doanh thu tăng, quản lí mới muốn tối ưu hóa doanh thu.

&nbsp; Vấn đề kinh doanh: Doanh thu tháng 2 bị giảm, Có những món gây lãng phí dẫn đến biên lợi nhuận bị thu hẹp. Quản lí cần xác định nguyên nhân: do kỹ năng tư vấn của nhân viên, do danh mục sản phẩm không phù hợp hay do nguyên nhân khác?" để tìm cách để tăng doanh thu.



###### **4.Data Processing (ETL)**:

1.Extract: Kết nối file Excel từ Kaggle

**A. Data Cleaning:**

&nbsp;  Xử lý Null/Missing Value: Null/Missing Value ở các cột quan trọng là 0%.

&nbsp;  Định dạng kiểu dữ liệu (Data Type): Chuyển transaction\_date sang Date và transaction\_time sang Time. Nếu để dạng Text, sẽ dễ thực hiện các phép tính Time Intelligence (WoW).

**B. Data Transformation:**

&nbsp;  Custom Column: Sử dụng quantity \* unit\_price để tạo cột Revenue

&nbsp;  Tạo cột category\_id, tạo cột hour dựa vào transaction\_time.

**C. Data Warehousing \& Data Modeling**: 

&nbsp;  Normalization: 

Từ một bảng phẳng (Nguon) duy nhất, thực hiện Reference tách thành các bảng Dimension: Dim\_date, Dim\_product, Dim\_store, Dim\_time và Fact: Fact\_sales.

Khi bảng gốc thay đổi, tất cả các bảng Dim sẽ tự động cập nhật theo, đảm bảo tính nhất quán của dữ liệu.

Loại bỏ dữ liệu trùng lặp : Tại các bảng Dim, giữ lại các giá trị duy nhất (Unique IDs) để đảm bảo mối quan hệ 1-nhiều với bảng Fact.

&nbsp;  Star Schema và Snowflake Schema:

Star Schema: Đặt Fact\_Sales ở giữa, bao quanh là các bảng Dim\_Date, Dim\_Store, Dim\_Time. Tối ưu hóa bộ nhớ và tốc độ lọc.

Snowflake Schema: Riêng với phần sản phẩm, tách thêm Dim\_Category từ Dim\_Product. Giúp chuẩn hóa dữ liệu.





###### **3. DAX Showcase**

1: Tổng doanh thu (Total Revenue)

Tính tổng số tiền kiếm được.

Total Revenue = SUM(Fact\_Sales\[Revenue])



2: Giá trị trung bình mỗi đơn (Avg Order Value - AOV)

Khách hàng chi bao nhiêu tiền cho mỗi lần ghé chi nhánh

Avg Order Value = DIVIDE(\[Total Revenue], \[Total Orders], 0)



3: Số lượng món trung bình mỗi đơn (Items per Order)

Khách thường mua 1 món hay mua kèm thêm món khác

Items per Order = DIVIDE(\[Total Quantity], \[Total Orders], 0)



4.Doanh thu tháng trước (Revenue Last Month)

Revenue Last Month = CALCULATE( \[Total revenue], DATEADD('Dim\_Date'\[Date], -1, MONTH))



5.Tỷ lệ gắn kèm bánh %:

Orders with Both = VAR OrdersWithMain = VALUES('Fact\_Sales'\[transaction\_id]) VAR FilteredOrders = CALCULATETABLE(OrdersWithMain,'Dim\_Product'\[product\_category] IN {"Coffee", "Tea"})

RETURN

CALCULATE(DISTINCTCOUNT('Fact\_Sales'\[transaction\_id]),'Dim\_Product'\[product\_category] = "Bakery",KEEPFILTERS(FilteredOrders))



Orders with CoffeeTea = VAR OrdersWithMain = CALCULATETABLE(VALUES('Fact\_Sales'\[transaction\_id]),'Dim\_Product'\[product\_category] IN {"Coffee", "Tea"})

RETURN

COUNTROWS(OrdersWithMain)

Bakery Attach Rate % = DIVIDE(\[Orders with Both], \[Orders with CoffeeTea], 0)



###### **4. Actionable Insights:**

Xu hướng: Đường dốc lên 

Doanh thu tăng liên tục qua các tháng. Việc tăng trưởng cho thấy chuỗi cafe không chỉ may mắn trong 1 khoảng thời gian ngắn, mà đang có một lượng khách hàng trung thành tăng dần hoặc quy mô vận hành đang mở rộng.



Insight 1:

Có khoảng 6 sản phẩm bán ít với đóng góp chưa tới 1% tổng lượng giao dịch theo từng chi nhánh. Việc duy trì các món này gây lãng phí nguyên liệu và làm thực khách tốn thêm thời gian đọc thực đơn, làm chậm tốc độ gọi món.

Hành động:

    Giai đoạn 1: Chạy chương trình "Sản phẩm giới hạn" cho các món này để xả kho nguyên liệu.

    Giai đoạn 2: Loại bỏ hoàn toàn khỏi thực đơn chính thức. Thay thế bằng các món Bán chạy, lãi cao.



Insight 2.1:

Tổng doanh thu hầu hết là từ items giá <5$ ở cả 3 chi nhánh. Vì đơn giá thấp (dưới $5), mỗi giao dịch chỉ cần tăng thêm 1 món phụ (như bánh $3) sẽ giúp doanh nghiệp tăng trưởng doanh thu tới 60-80% trên mỗi hóa đơn."

Hành động: Thiết kế các gói phần bổ sung ngay tại quầy thu ngân. Ví dụ: "Thêm 1 bánh quy chỉ với $1.5". Với các sản phẩm giá thấp, khách hàng ít đắn đo hơn khi quyết định mua thêm.



Insight 2.2: 

Tổng doanh thu hầu hết là từ items giá <5$ ở cả 3 chi nhánh. Lợi nhuận nằm ở việc quản lý từng mililit sữa và từng gram cafe. 

Hành động: Triển khai hệ thống định lượng chính xác tại cả 3 chi nhánh. Vì doanh thu đến từ số lượng lớn, một sai lệch nhỏ (ví dụ thừa 10ml sữa/ly) nhân với hàng nghìn ly mỗi ngày sẽ tạo ra khoản thất thoát khổng lồ vào cuối tháng.



Insight 3:

Attach Rate(Tỷ lệ đính kèm) của Bakery chỉ khoảng 22%. Tăng Attach Rate thêm 5% có thể giúp lợi nhuận của quán tăng trưởng đáng kể nhờ cộng thêm giá trị bánh.

Hành động:  Cách Upsell: Coffee thường có biên lợi nhuận thấp hơn đồ ăn. Hướng dẫn nhân viên mời chào bánh hiệu quả hơn và đổi menu bánh hấp dẫn với người uống coffee.

Chi nhánh Ho Chi Minh có Attach Rate cao hơn Ha noi và Da Nang. Do cách bày trí tủ bánh tại đó dễ thấy hơn.



Insight 4:

Doanh thu tăng trưởng liên tục từ tháng 1 đến tháng 6 bị sụt giảm vào tháng 2. 

Số ngày kinh doanh ngắn nhất: Tháng 2 chỉ có 28 ngày. So với tháng 1 (31 ngày) hay tháng 3 (31 ngày), mất 3 ngày doanh thu (khoảng 10% của tháng). Hiệu ứng sau lễ : Sau đợt chi tiêu mạnh vào tháng 1 do năm mới, khách hàng thường có xu hướng tiết kiệm và ăn uống lành mạnh hơn vào tháng 2.

Hành động: Vì tháng 2 ngắn ngày, quán cần các chiến dịch tăng doanh thu mạnh trong thời gian ngắn như Valentine's Day (14/2) để bù lại khoảng trống. 

Quản lý tồn kho: Dự báo sự sụt giảm vào tháng 2 giúp chủ động giảm đặt nguyên liệu tươi bánh/sữa để tránh lãng phí .

Duy trì đà tăng trưởng: Từ tháng 3 trở đi, doanh thu phục hồi và tăng mạnh mẽ, chứng tỏ khách trung thành vẫn duy trì và chiến lược đang đi đúng hướng.



Insight 5:

Nhóm Coffee và Tea chiếm hơn 50% doanh thu. Đây là lý do chính khiến khách hàng mua hàng ở cả 3 chi nhánh.

Dù đơn giá trung bình thường dưới $5, số lượng giao dịch lớn, nhóm này bù đắp toàn bộ chi phí vận hành, nhân viên cho chi nhánh.

Tối ưu hóa: Vì đây là nhóm chủ lực, bất kỳ sự chậm trễ nào trong pha chế Coffee/Tea cũng sẽ làm nghẽn hệ thống.

Hành động: Sắp xếp quầy bar đảm bảo các nguyên liệu cho Coffee/Tea luôn ở vị trí thuận lợi nhất. Triển khai chương trình Bánh đồng giá $2 khi mua kèm bất kỳ ly Coffee/Tea. Tận dụng lượng khách lớn của nhóm đồ uống để giải quyết hàng tồn kho của nhóm bánh.















































































