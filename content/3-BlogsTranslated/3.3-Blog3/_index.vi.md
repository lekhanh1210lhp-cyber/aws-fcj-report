---
title: "Blog 3"
date: "2025-04-09"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# **Xử lý hàng triệu sự kiện observability với Apache Flink và ghi trực tiếp vào Prometheus**

_bởi Lorenzo Nicora và Francisco Morillo vào ngày 09 THÁNG 4 NĂM 2025 trong Amazon Managed Grafana, Amazon Managed Service for Apache Flink, Amazon Managed Service for Prometheus, Analytics, Internet of Things, Open Source_

AWS gần đây đã công bố hỗ trợ một trình kết nối (connector) Apache Flink mới cho Prometheus. Trình kết nối mới này, được AWS đóng góp cho dự án mã nguồn mở Flink, bổ sung Prometheus và Amazon Managed Service for Prometheus làm điểm đến mới cho Flink.

Trong bài viết này, chúng tôi sẽ giải thích cách hoạt động của trình kết nối mới. Chúng tôi cũng sẽ chỉ cho bạn cách quản lý lượng biến thể dữ liệu (cardinality) của metrics Prometheus bằng cách tiền xử lý (preprocessing) dữ liệu thô với Flink để xây dựng khả năng quan sát (observability) thời gian thực bằng Amazon Managed Service for Prometheus và Amazon Managed Grafana.

Amazon Managed Service for Prometheus là một dịch vụ giám sát bảo mật, phi máy chủ (serverless), có khả năng mở rộng và tương thích với Prometheus. Bạn có thể sử dụng cùng một mô hình dữ liệu Prometheus mã nguồn mở và ngôn ngữ truy vấn mà bạn đang dùng hiện nay để giám sát hiệu suất của các khối lượng công việc (workloads) mà không cần phải quản lý cơ sở hạ tầng bên dưới. Trình kết nối Flink là các thành phần phần mềm giúp di chuyển dữ liệu vào và ra khỏi ứng dụng Amazon Managed Service for Apache Flink. Bạn có thể sử dụng trình kết nối mới này để gửi dữ liệu đã xử lý đến đích là Amazon Managed Service for Prometheus bắt đầu từ phiên bản Flink 1.19. Với Amazon Managed Service for Apache Flink, bạn có thể chuyển đổi và phân tích dữ liệu theo thời gian thực. Không có máy chủ và cụm (clusters) nào để quản lý, cũng như không có cơ sở hạ tầng điện toán và lưu trữ nào cần thiết lập.

## **Khả năng quan sát vượt ra ngoài tài nguyên máy tính**
Trong một thế giới ngày càng kết nối, ranh giới của các hệ thống mở rộng ra ngoài các tài nguyên máy tính (compute assets), cơ sở hạ tầng CNTT và ứng dụng. Các tài sản phân tán như thiết bị Internet of Things (IoT), ô tô được kết nối và các thiết bị truyền phát phương tiện của người dùng cuối là một phần không thể thiếu trong hoạt động kinh doanh ở nhiều lĩnh vực. Khả năng quan sát mọi tài sản của doanh nghiệp là chìa khóa để phát hiện sớm các vấn đề tiềm ẩn, cải thiện trải nghiệm của khách hàng và bảo vệ lợi nhuận của doanh nghiệp.

## **Số liệu (Metrics) và chuỗi thời gian (time series)**
Sẽ rất hữu ích khi coi observability bao gồm ba trụ cột: metrics (số liệu), logs (nhật ký) và traces (dấu vết). Trụ cột phù hợp nhất đối với các thiết bị phân tán, như IoT, là metrics. Điều này là do metrics có thể ghi lại các phép đo từ các cảm biến hoặc đếm số lượng các sự kiện cụ thể do thiết bị phát ra.

Metrics là một chuỗi các mẫu của một phép đo nhất định tại những thời điểm cụ thể. Ví dụ, trong trường hợp một chiếc xe được kết nối, chúng có thể là thông số từ cảm biến vòng quay (RPM) của động cơ điện. Metrics thường được biểu diễn dưới dạng chuỗi thời gian (time series), hoặc các chuỗi điểm dữ liệu rời rạc theo trình tự thời gian. Chuỗi thời gian của metrics thường gắn liền với các chiều (dimensions), còn được gọi là nhãn (labels) hoặc thẻ (tags), để giúp phân loại và phân tích dữ liệu. Trong trường hợp một chiếc xe được kết nối, các nhãn có thể như sau:
*   **Tên metric (Metric name)** – Ví dụ: “Vòng quay động cơ điện (Electric Motor RPM)”
*   **ID xe (Vehicle ID)** – Một mã định danh duy nhất của chiếc xe, như Số nhận dạng xe (VIN)

## **Prometheus với tư cách là một cơ sở dữ liệu chuỗi thời gian chuyên dụng**
Prometheus là một giải pháp phổ biến để lưu trữ và phân tích metrics. Prometheus định nghĩa một giao diện tiêu chuẩn để lưu trữ và truy vấn chuỗi thời gian. Thường được sử dụng kết hợp với các công cụ trực quan hóa như Grafana, Prometheus được tối ưu hóa cho các bảng điều khiển (dashboards) và cảnh báo theo thời gian thực.

Thường được xem xét chủ yếu để quan sát các tài nguyên máy tính, như container hoặc ứng dụng, Prometheus thực sự là một cơ sở dữ liệu chuỗi thời gian chuyên dụng có thể được sử dụng hiệu quả để quan sát các loại tài sản phân tán khác nhau, bao gồm cả thiết bị IoT.

Amazon Managed Service for Prometheus là một dịch vụ giám sát serverless, tương thích với Prometheus. Xem Amazon Managed Service for Prometheus là gì? để tìm hiểu thêm về dịch vụ này.

## **Xử lý hiệu quả các sự kiện observability ở quy mô lớn**
Việc xử lý dữ liệu observability ở quy mô lớn ngày càng trở nên khó khăn hơn, do số lượng tài sản và các metrics độc nhất, đặc biệt là khi quan sát các thiết bị được phân tán khổng lồ, vì những lý do sau:
*   **Độ đa dạng cao (High cardinality)** – Mỗi thiết bị phát ra nhiều metrics hoặc loại sự kiện khác nhau, mỗi loại được theo dõi độc lập.
*   **Tần suất cao** – Các thiết bị có thể phát ra các sự kiện rất thường xuyên, nhiều lần trong một giây. Điều này có thể dẫn đến một khối lượng lớn dữ liệu thô. Khía cạnh này đặc biệt thể hiện sự khác biệt chính so với việc quan sát các tài nguyên máy tính, vốn thường được thu thập (scraped) ở các khoảng thời gian dài hơn.
*   **Sự kiện đến vào những khoảng thời gian không đều và không theo thứ tự** – Không giống như tài nguyên tính toán thường được thu thập ở các khoảng thời gian đều đặn, chúng ta thường thấy sự chậm trễ trong việc truyền tải hoặc các thiết bị tạm thời bị ngắt kết nối, khiến các sự kiện đến ở những khoảng thời gian không đều. Các sự kiện đồng thời từ các thiết bị khác nhau có thể đi theo các đường dẫn khác nhau và đến vào những thời điểm khác nhau.
*   **Thiếu thông tin ngữ cảnh** – Các thiết bị thường truyền tải dữ liệu qua các kênh có băng thông hạn chế, chẳng hạn như GPRS hoặc Bluetooth. Để tối ưu hóa việc liên lạc, các sự kiện hiếm khi chứa thông tin ngữ cảnh, chẳng hạn như kiểu thiết bị hoặc chi tiết người dùng. Tuy nhiên, thông tin này lại cần thiết cho một quá trình observability hiệu quả.
*   **Tính toán metrics từ các sự kiện** – Các thiết bị thường phát ra các sự kiện cụ thể khi có sự việc xảy ra. Ví dụ, khi bộ phận đánh lửa của xe được bật hoặc tắt, hoặc khi máy tính trên xe phát ra một cảnh báo. Đây không phải là các metrics trực tiếp. Tuy nhiên, việc đếm và đo lường tỷ lệ của các sự kiện này là những metrics có giá trị có thể được suy luận từ các sự kiện đó.

Việc trích xuất hiệu quả giá trị từ các sự kiện thô đòi hỏi phải xử lý. Quá trình xử lý có thể diễn ra khi đọc (on read), lúc bạn truy vấn dữ liệu, hoặc thực hiện từ trước (upfront) trước khi lưu trữ.

## **Lưu trữ và phân tích các sự kiện thô**
Phương pháp phổ biến đối với các sự kiện observability, và cụ thể là với metrics, là "lưu trữ trước" (storing first). Bạn có thể chỉ cần ghi các metrics thô vào Prometheus. Quá trình xử lý, chẳng hạn như nhóm, tổng hợp và tính toán các metrics phái sinh, diễn ra "khi truy vấn" (on query), lúc dữ liệu được trích xuất từ Prometheus.

Phương pháp này có thể trở nên đặc biệt kém hiệu quả khi bạn đang xây dựng các bảng điều khiển hoặc cảnh báo thời gian thực và dữ liệu của bạn có độ đa dạng cao hoặc tần suất cao. Khi một cơ sở dữ liệu chuỗi thời gian liên tục được truy vấn, một lượng lớn dữ liệu sẽ liên tục được trích xuất từ bộ lưu trữ và được xử lý. Sơ đồ sau minh họa quy trình làm việc này.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_1.jpg)

## **Tiền xử lý các sự kiện observability thô**
Việc tiền xử lý các sự kiện thô trước khi lưu trữ sẽ chuyển dời khối lượng công việc về phía trước (shifts the work left), như được minh họa trong sơ đồ sau. Điều này làm tăng hiệu quả của các bảng điều khiển và cảnh báo thời gian thực, cho phép giải pháp có khả năng mở rộng.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_2.jpg)

## **Apache Flink cho việc tiền xử lý sự kiện observability**
Việc tiền xử lý các sự kiện observability thô yêu cầu một công cụ xử lý cho phép bạn thực hiện những việc sau:
*   **Làm phong phú (Enrich) các sự kiện một cách hiệu quả**, tra cứu dữ liệu tham chiếu và thêm các chiều (dimensions) mới vào các sự kiện thô. Ví dụ, thêm kiểu xe dựa trên ID xe. Việc làm phong phú cho phép thêm các chiều mới vào chuỗi thời gian, giúp bạn có thể phân tích những trường hợp vốn dĩ không thể thực hiện.
*   **Tổng hợp các sự kiện thô qua các khung thời gian (time windows)**, để giảm tần suất. Ví dụ, nếu một chiếc xe phát ra phép đo nhiệt độ động cơ mỗi giây, bạn có thể xuất ra một mẫu duy nhất với mức trung bình trong 5 giây. Prometheus có thể tổng hợp hiệu quả các mẫu có tần suất cao khi đọc. Tuy nhiên, việc đưa dữ liệu vào với tần suất cao hơn nhiều so với mức hữu ích cho bảng điều khiển và cảnh báo thời gian thực không phải là cách sử dụng hiệu quả thông lượng (throughput) và lưu trữ của Prometheus.
*   **Tổng hợp các sự kiện thô qua các chiều (dimensions)**, để giảm độ đa dạng (cardinality). Ví dụ: tổng hợp một số phép đo theo từng kiểu xe.
*   **Tính toán các metrics phái sinh thông qua việc áp dụng logic tùy ý.** Ví dụ, đếm số lượng sự kiện cảnh báo được phát ra từ mỗi chiếc xe. Điều này cũng cho phép thực hiện phân tích mà nếu chỉ dùng Prometheus và Grafana thì không thể.
*   **Hỗ trợ ngữ nghĩa thời gian sự kiện (event-time semantics)**, để tổng hợp qua thời gian các sự kiện từ nhiều nguồn khác nhau.

Một công cụ tiền xử lý như vậy cũng phải có khả năng mở rộng và xử lý khối lượng lớn các sự kiện thô đầu vào, đồng thời xử lý dữ liệu với độ trễ thấp—thường là dưới một giây (subsecond) hoặc một vài giây—để kích hoạt các bảng điều khiển và cảnh báo theo thời gian thực. Để giải quyết các yêu cầu này, chúng tôi thấy nhiều khách hàng đang sử dụng Flink.

Apache Flink đáp ứng các yêu cầu nói trên. Flink là một framework và công cụ xử lý luồng phân tán (distributed stream processing engine), được thiết kế để thực hiện các tính toán ở tốc độ bộ nhớ trong (in-memory) và ở quy mô lớn. Amazon Managed Service for Apache Flink mang đến trải nghiệm phi máy chủ, được quản lý hoàn toàn, cho phép bạn chạy các ứng dụng Flink của mình mà không cần quản lý cơ sở hạ tầng hoặc các cụm (clusters).

Amazon Managed Service for Apache Flink có thể xử lý các sự kiện thô được đưa vào. Các metrics kết quả, với độ đa dạng và tần suất thấp hơn, cùng với các dimensions bổ sung, có thể được ghi vào Prometheus để trực quan hóa và phân tích hiệu quả hơn. Sơ đồ sau minh họa quy trình làm việc này.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_3.jpg)

## **Tích hợp Apache Flink và Prometheus**
Trình kết nối (connector) Flink Prometheus mới cho phép các ứng dụng Flink ghi dữ liệu chuỗi thời gian đã được tiền xử lý vào Prometheus một cách liền mạch. Không cần thành phần trung gian nào và không có yêu cầu phải triển khai tích hợp tùy chỉnh. Trình kết nối này được thiết kế để mở rộng quy mô, sử dụng khả năng mở rộng theo chiều ngang của Flink và tối ưu hóa việc ghi dữ liệu vào backend của Prometheus thông qua giao diện Remote-Write.

## **Ví dụ về trường hợp sử dụng**
AnyCompany là một công ty cho thuê ô tô quản lý một đội xe gồm hàng trăm nghìn chiếc xe hybrid (lai) được kết nối, ở nhiều khu vực. Mỗi chiếc xe liên tục truyền các phép đo từ nhiều cảm biến. Mỗi cảm biến phát ra một mẫu dữ liệu mỗi giây hoặc thường xuyên hơn. Các phương tiện cũng truyền đi các sự kiện cảnh báo khi máy tính trên xe phát hiện có sự cố. Sơ đồ sau minh họa quy trình làm việc này.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_4.jpg)

AnyCompany đang có kế hoạch sử dụng Amazon Managed Service for Prometheus và Amazon Managed Grafana để trực quan hóa các metrics của xe và thiết lập các cảnh báo tùy chỉnh.

Tuy nhiên, việc xây dựng một bảng điều khiển thời gian thực dựa trên dữ liệu thô do các phương tiện truyền tới có thể sẽ phức tạp và kém hiệu quả. Mỗi chiếc xe có thể có hàng trăm cảm biến, mỗi cảm biến dẫn đến một chuỗi thời gian riêng biệt để hiển thị. Ngoài ra, AnyCompany muốn theo dõi hành vi của các kiểu xe khác nhau. Đáng tiếc, các sự kiện do phương tiện truyền đi chỉ chứa số VIN. Kiểu xe có thể được suy ra bằng cách tra cứu (kết hợp) với một số dữ liệu tham chiếu.

Để vượt qua những thách thức này, AnyCompany đã xây dựng một quy trình tiền xử lý dựa trên Amazon Managed Service for Apache Flink. Quá trình này có các khả năng sau:
*   Làm phong phú dữ liệu thô bằng cách thêm kiểu xe và tra cứu dữ liệu tham chiếu dựa trên ID của phương tiện.
*   Giảm độ đa dạng (cardinality) bằng cách tổng hợp kết quả theo từng kiểu xe, khả dụng sau bước làm phong phú.
*   Giảm tần suất của các metrics thô để giảm băng thông ghi, tổng hợp trong các khung thời gian vài giây.
*   Tính toán các metrics phái sinh dựa trên nhiều metrics thô. Ví dụ, xác định xem xe có đang di chuyển hay không khi động cơ đốt trong (IC) hoặc động cơ điện đang quay.

Kết quả của việc tiền xử lý là các metrics có tính ứng dụng (actionable) cao hơn. Ví dụ, một bảng điều khiển được xây dựng trên các metrics này có thể giúp xác định xem bản cập nhật phần mềm mới nhất được phát hành qua mạng (over-the-air) cho tất cả các phương tiện của một kiểu xe cụ thể ở các khu vực cụ thể, có gây ra sự cố hay không.

Sử dụng trình kết nối Flink Prometheus, ứng dụng tiền xử lý có thể ghi trực tiếp vào Amazon Managed Service for Prometheus mà không cần các thành phần trung gian.

Không có điều gì ngăn bạn chọn việc ghi các metrics thô với đầy đủ độ đa dạng và tần suất vào Prometheus, cho phép bạn đi sâu (drill down) phân tích chi tiết từng chiếc xe. Trình kết nối Flink Prometheus được thiết kế để mở rộng quy mô bằng cách gộp nhóm (batching) và song song hóa (parallelizing) các tác vụ ghi.

## **Tổng quan về giải pháp**
Kho lưu trữ GitHub sau đây chứa một ví dụ hoàn chỉnh (end-to-end) hư cấu bao gồm trường hợp sử dụng này. Sơ đồ sau minh họa kiến trúc của ví dụ này.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_5.jpg)

Quy trình làm việc bao gồm các bước sau:
1. Các phương tiện, đường truyền vô tuyến và việc tiếp nhận các sự kiện IoT đã được trừu tượng hóa (abstracted away) và được thay thế bằng một trình tạo dữ liệu (data generator) để tạo ra các sự kiện thô cho một trăm nghìn phương tiện hư cấu. Để đơn giản, bản thân trình tạo dữ liệu này là một ứng dụng Amazon Managed Service for Apache Flink.
2. Các sự kiện xe thô được gửi đến một dịch vụ lưu trữ luồng (stream storage service). Trong ví dụ này, chúng tôi sử dụng Amazon Managed Streaming for Apache Kafka (Amazon MSK).
3. Cốt lõi của hệ thống là ứng dụng tiền xử lý (preprocessor), chạy trên Amazon Managed Service for Apache Flink. Chúng ta sẽ đi sâu hơn vào chi tiết của bộ xử lý trong các phần tiếp theo.
4. Các metrics đã xử lý được ghi trực tiếp vào backend của Prometheus, trong Amazon Managed Service for Prometheus.
5. Các metrics được sử dụng để tạo các bảng điều khiển thời gian thực trên Amazon Managed Grafana.

Ảnh chụp màn hình sau đây cho thấy một bảng điều khiển mẫu.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_6.jpg)

## **Các sự kiện phương tiện thô**
Mỗi chiếc xe truyền ba metric gần như mỗi giây:
*   Vòng quay động cơ đốt trong (IC)
*   Vòng quay động cơ điện
*   Số lượng cảnh báo được báo cáo

Các sự kiện thô được xác định bởi ID xe và khu vực nơi chiếc xe đang ở.

## **Ứng dụng tiền xử lý (Preprocessor application)**
Sơ đồ sau minh họa luồng logic của ứng dụng tiền xử lý chạy trên Amazon Managed Service for Apache Flink.

## ![](/images/3-BlogsTranslated/3.3-Blog3/image_7.jpg)

Quy trình làm việc bao gồm các bước sau:
1. Các sự kiện thô được lấy từ Amazon MSK thông qua nguồn Flink Kafka.
2. Một toán tử làm phong phú (enrichment operator) sẽ thêm thông tin kiểu xe, vốn không có trong các sự kiện thô. Dimension bổ sung này sau đó được sử dụng để tổng hợp các sự kiện thô. Các metrics kết quả chỉ có hai dimensions: kiểu xe và khu vực.
3. Các sự kiện thô sau đó được tổng hợp qua các khung thời gian (5 giây) để giảm tần suất. Trong ví dụ này, logic tổng hợp cũng tạo ra một metric phái sinh: số lượng xe đang chuyển động. Một metric mới có thể được lấy từ các metric thô thông qua các logic tùy ý. Để làm ví dụ, một chiếc xe được coi là "đang chuyển động" nếu thông số vòng quay động cơ IC hoặc động cơ điện khác 0.
4. Các metrics đã xử lý được ánh xạ vào cấu trúc dữ liệu đầu vào của trình kết nối Flink Prometheus, cấu trúc này ánh xạ trực tiếp đến các bản ghi chuỗi thời gian mà giao diện Prometheus Remote-Write mong đợi. Tham khảo tài liệu của trình kết nối để biết thêm chi tiết.
5. Cuối cùng, các metrics được gửi đến Prometheus thông qua trình kết nối Flink Prometheus. Việc xác thực tác vụ ghi (write authentication), theo yêu cầu của Amazon Managed Service for Prometheus, được kích hoạt một cách liền mạch bằng cách sử dụng bộ ký yêu cầu (request signer) của Amazon Managed Service for Prometheus được cung cấp kèm theo trình kết nối. Thông tin xác thực tự động được lấy từ vai trò AWS Identity and Access Management (IAM) của ứng dụng Amazon Managed Service for Apache Flink. Không cần thông tin bảo mật (secret) hoặc thông tin đăng nhập (credential) bổ sung nào.

Trong kho lưu trữ GitHub, bạn có thể tìm thấy hướng dẫn từng bước để thiết lập ví dụ thực tế này và tạo bảng điều khiển Grafana.

## **Các tính năng chính của trình kết nối Flink Prometheus**
Trình kết nối Flink Prometheus cho phép các ứng dụng Flink ghi các metrics đã xử lý vào Prometheus bằng giao diện Remote-Write.

Trình kết nối được thiết kế để mở rộng thông lượng ghi (write throughput) bằng cách:
*   Song song hóa các bản ghi, sử dụng khả năng xử lý song song (parallelism) của Flink
*   Gộp nhóm (Batching) nhiều mẫu trong một yêu cầu ghi duy nhất đến Prometheus endpoint

Việc xử lý lỗi tuân thủ theo quy chuẩn Prometheus Remote-Write 1.0. Các quy định kỹ thuật này đặc biệt nghiêm ngặt về việc Prometheus từ chối dữ liệu bị sai định dạng hoặc không theo thứ tự.

Khi một bản ghi bị sai định dạng hoặc không theo thứ tự bị từ chối, trình kết nối sẽ loại bỏ yêu cầu ghi vi phạm đó và tiếp tục, ưu tiên độ mới của dữ liệu (data freshness) hơn là tính toàn vẹn của dữ liệu (completeness). Tuy nhiên, trình kết nối làm cho việc mất dữ liệu này có thể quan sát được (observable), bằng cách phát ra các mục nhật ký WARN và hiển thị các metrics đo lường khối lượng dữ liệu bị loại bỏ. Trong Amazon Managed Service for Apache Flink, các metrics của trình kết nối này có thể được tự động xuất sang Amazon CloudWatch.

## **Trách nhiệm của người dùng**
Trình kết nối được tối ưu hóa cho tính hiệu quả, thông lượng ghi và độ trễ. Việc xác thực (Validation) dữ liệu đầu vào sẽ đặc biệt tốn kém về mức sử dụng CPU. Ngoài ra, các triển khai backend Prometheus khác nhau thực thi các ràng buộc một cách khác nhau. Vì những lý do này, trình kết nối không xác thực dữ liệu đầu vào trước khi ghi vào Prometheus.

Người dùng có trách nhiệm đảm bảo rằng dữ liệu được gửi đến trình kết nối Flink Prometheus tuân theo các ràng buộc được thực thi bởi các bản triển khai Prometheus cụ thể mà họ đang sử dụng.

## **Thứ tự (Ordering)**
Thứ tự là yếu tố đặc biệt có liên quan. Prometheus mong đợi rằng các mẫu thuộc cùng một chuỗi thời gian—các mẫu có cùng tên metric và nhãn—phải được ghi theo thứ tự thời gian. Trình kết nối đảm bảo thứ tự này không bị mất khi dữ liệu được phân vùng để thực hiện ghi song song.

Tuy nhiên, người dùng có trách nhiệm duy trì thứ tự từ nguồn (upstream) trong pipeline. Để đạt được điều này, người dùng phải thiết kế việc phân vùng dữ liệu bên trong ứng dụng Flink và dịch vụ lưu trữ luồng một cách cẩn thận. Chỉ sử dụng chức năng phân vùng theo khóa (partitioning by key) và các khóa phân vùng phải bao hàm tên metric và tất cả các nhãn sẽ được sử dụng trong Prometheus.

## **Kết luận**
Prometheus là một cơ sở dữ liệu chuỗi thời gian chuyên dụng, được thiết kế để xây dựng các bảng điều khiển và cảnh báo theo thời gian thực. Amazon Managed Service for Prometheus là một backend được quản lý hoàn toàn, phi máy chủ (serverless) tương thích với tiêu chuẩn mã nguồn mở Prometheus. Amazon Managed Grafana cho phép bạn xây dựng các bảng điều khiển thời gian thực, giao tiếp liền mạch với Amazon Managed Service for Prometheus.

Bạn có thể sử dụng Prometheus cho các trường hợp sử dụng về observability vượt ra khỏi các tài nguyên máy tính, để quan sát các thiết bị IoT, ô tô được kết nối, thiết bị truyền phát đa phương tiện và các tài sản phân tán cao độ khác cung cấp dữ liệu từ xa (telemetry).

Việc trực tiếp trực quan hóa và phân tích dữ liệu có độ đa dạng (cardinality) và tần suất cao có thể sẽ không hiệu quả. Tiền xử lý các sự kiện observability thô với Amazon Managed Service for Apache Flink sẽ dịch chuyển công việc sang trái (shifts the work left), đơn giản hóa đáng kể các bảng điều khiển hoặc cảnh báo mà bạn có thể xây dựng trên Amazon Managed Service for Prometheus.

Để biết thêm thông tin về việc chạy Flink, Prometheus và Grafana trên AWS, hãy xem tài liệu của các dịch vụ này:
*   Amazon Managed Service for Apache Flink
*   Amazon Managed Service for Prometheus
*   Amazon Managed Grafana

Để biết thêm thông tin về việc tích hợp Flink Prometheus, hãy xem tài liệu của Apache Flink.

**Bài đăng trên AWS Study Group**
## ![](/images/image_3.png)

## **Về các tác giả**

<div style="display:flex; flex-direction:column; gap:1rem;">

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.3-Blog3/image_8.jpg" alt="Lorenzo Nicora" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Lorenzo Nicora</strong><br/>
      Lorenzo Nicora làm việc với tư cách là Kiến trúc sư Giải pháp Luồng cao cấp (Senior Streaming Solution Architect) tại AWS, giúp đỡ khách hàng trên khắp khu vực EMEA. Anh đã dành hơn 25 năm xây dựng các hệ thống chuyên sâu về dữ liệu, tập trung vào đám mây, làm việc trong nhiều ngành công nghiệp thông qua cả các công ty tư vấn và công ty sản phẩm. Anh đã sử dụng rộng rãi các công nghệ mã nguồn mở và đóng góp cho một số dự án, bao gồm Apache Flink, đồng thời là người duy trì trình kết nối Flink Prometheus.
    </div>
  </div>

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.3-Blog3/image_9.jpg" alt="Francisco Morillo" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Francisco Morillo</strong><br/>
      Francisco Morillo là một Kiến trúc sư Giải pháp Luồng (Streaming Solutions Architect) cao cấp tại AWS. Francisco làm việc với các khách hàng của AWS, giúp họ thiết kế kiến trúc phân tích thời gian thực bằng các dịch vụ của AWS, hỗ trợ Amazon MSK và Amazon Managed Service for Apache Flink. Anh cũng là một cộng tác viên chính cho trình kết nối Flink Prometheus.
    </div>
  </div>

</div>