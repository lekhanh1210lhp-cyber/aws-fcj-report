---
title: "Blog 1"
date: "2025-09-09"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# **Cách PostNL xử lý hàng tỷ sự kiện IoT với Amazon Managed Service cho Apache Flink**

_bởi Çağrı Çakır, Ozge Kavalci, Amit Singh, và Lorenzo Nicora vào ngày 15 THÁNG 7 NĂM 2024 trong [Amazon Managed Service cho Apache Flink](https://aws.amazon.com/vi/blogs/big-data/category/analytics/amazon-managed-service-for-apache-flink/), [Phân tích (Analytics)](https://aws.amazon.com/vi/blogs/big-data/category/analytics/), [Giải pháp Khách hàng](https://aws.amazon.com/vi/blogs/big-data/category/post-types/customer-solutions/), [Internet vạn vật (IoT)](https://aws.amazon.com/vi/blogs/big-data/category/internet-of-things/), [Kinesis Data Analytics](https://aws.amazon.com/vi/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-analytics/)_

**Bài viết này được đồng chắp bút bởi Çağrı Çakır và Özge Kavalcı từ PostNL.**

[**PostNL**](https://www.postnl.nl/en/) là nhà cung cấp dịch vụ bưu chính phổ cập được chỉ định cho Hà Lan và có ba đơn vị kinh doanh chính cung cấp dịch vụ chuyển phát thư, chuyển phát bưu kiện và các giải pháp logistics cho thương mại điện tử cũng như các giải pháp xuyên biên giới. Với 5.800 điểm bán lẻ, 11.000 hộp thư và hơn 900 tủ đựng bưu kiện tự động, công ty đóng vai trò quan trọng trong chuỗi giá trị logistics. Mục tiêu của họ là trở thành tổ chức giao hàng được ưu tiên lựa chọn bằng cách làm cho việc gửi và nhận bưu kiện cũng như thư từ trở nên dễ dàng nhất có thể. Với gần 34.000 nhân viên, PostNL là trung tâm của xã hội. Vào một ngày làm việc điển hình, công ty giao trung bình 1,1 triệu bưu kiện và 6,9 triệu lá thư trên khắp Bỉ, Hà Lan và Luxembourg.

Trong bài viết này, chúng tôi mô tả giải pháp xử lý luồng kế thừa (legacy) của PostNL, những thách thức của nó và lý do tại sao PostNL chọn [**Amazon Managed Service cho Apache Flink**](https://aws.amazon.com/managed-service-apache-flink/) để giúp hiện đại hóa nền tảng xử lý luồng dữ liệu Internet vạn vật (IoT) của họ. Chúng tôi cung cấp một kiến trúc tham chiếu, mô tả các bước chúng tôi đã thực hiện để di chuyển sang [**Apache Flink**](https://flink.apache.org/) và những bài học rút ra trong suốt quá trình này.

Với quá trình di chuyển này, PostNL đã có thể xây dựng một giải pháp xử lý luồng có khả năng mở rộng, mạnh mẽ và có thể mở rộng thêm cho nền tảng IoT của họ. Apache Flink là sự lựa chọn hoàn hảo cho IoT. Nhờ khả năng mở rộng theo chiều ngang (scaling horizontally), nó cho phép xử lý khối lượng dữ liệu khổng lồ do các thiết bị IoT tạo ra. Với ngữ nghĩa thời gian sự kiện (event time semantics), bạn có thể xử lý chính xác các sự kiện theo đúng thứ tự chúng được tạo ra, ngay cả từ những thiết bị thỉnh thoảng bị mất kết nối.

PostNL rất hào hứng với tiềm năng của Apache Flink và hiện đang có kế hoạch sử dụng Managed Service cho Apache Flink cho các trường hợp sử dụng dữ liệu luồng khác, cũng như chuyển đổi nhiều logic nghiệp vụ hơn lên hệ thống tuyến trên (upstream) vào Apache Flink.

## **Apache Flink và Managed Service cho Apache Flink**
Apache Flink là một framework tính toán phân tán cho phép xử lý dữ liệu thời gian thực có trạng thái (stateful real-time data processing). Nó cung cấp một bộ API duy nhất để xây dựng các tác vụ xử lý hàng loạt (batch) và xử lý luồng (streaming), giúp các nhà phát triển dễ dàng làm việc với luồng dữ liệu có giới hạn (bounded) và không có giới hạn (unbounded). Managed Service cho Apache Flink là một dịch vụ của AWS cung cấp hạ tầng serverless (phi máy chủ), được quản lý toàn diện để chạy các ứng dụng Apache Flink. Các nhà phát triển có thể xây dựng các ứng dụng Apache Flink có tính sẵn sàng cao, chịu lỗi tốt và dễ dàng mở rộng một cách dễ dàng mà không cần phải trở thành chuyên gia trong việc xây dựng, cấu hình và bảo trì các cụm Apache Flink trên AWS.

## **Thách thức của dữ liệu IoT thời gian thực ở quy mô lớn**
Ngày nay, nền tảng IoT của PostNL, [**giải pháp Roller Cages**](https://www.youtube.com/watch?v=oOtXz08OUec), theo dõi hơn 380.000 tài sản bằng công nghệ Bluetooth Năng lượng Thấp (BLE) trong thời gian gần thực (near real time). Nền tảng IoT được thiết kế để cung cấp thông tin về tính sẵn sàng, rào địa lý (geofencing) và sự kiện trạng thái cơ bản (bottom state) của từng tài sản bằng cách sử dụng dữ liệu cảm biến đo từ xa như điểm định vị GPS và gia tốc kế truyền từ các thiết bị Bluetooth. Các sự kiện đó được sử dụng bởi các nhóm tiêu thụ nội bộ khác nhau nhằm giúp hoạt động logistics trở nên dễ lập kế hoạch hơn, hiệu quả hơn và bền vững hơn.

## ![](/images/3-BlogsTranslated/3.1-Blog1/image_1.jpg) 

Việc theo dõi lượng lớn tài sản liên tục phát ra các số đo cảm biến khác nhau này chắc chắn sẽ tạo ra hàng tỷ sự kiện IoT thô cho nền tảng IoT cũng như cho các hệ thống hạ nguồn (downstream). Việc xử lý lặp đi lặp lại khối lượng tải này cả bên trong nền tảng IoT và xuyên suốt các hệ thống hạ nguồn đều không đem lại hiệu quả về chi phí và cũng không dễ bảo trì. Để giảm thiểu mức độ tập hợp (cardinality) của các sự kiện, nền tảng IoT sử dụng quá trình xử lý luồng để tổng hợp dữ liệu qua các khung cửa sổ thời gian cố định. Các quá trình tổng hợp này phải dựa trên thời điểm mà thiết bị phát ra sự kiện. Loại hình tổng hợp dựa trên thời gian sự kiện này trở nên phức tạp khi các sự kiện có thể bị chậm trễ và đến không theo thứ tự, điều này có thể thường xuyên xảy ra với các thiết bị IoT thỉnh thoảng bị ngắt kết nối tạm thời.

Biểu đồ sau đây minh họa luồng dữ liệu tổng thể từ biên (edge) đến các hệ thống hạ nguồn.

## ![](/images/3-BlogsTranslated/3.1-Blog1/image_2.jpg)

Quy trình làm việc bao gồm các thành phần sau:

1. Kiến trúc biên (edge architecture) bao gồm các thiết bị IoT BLE đóng vai trò là nguồn dữ liệu đo từ xa và các thiết bị cổng nối (gateway) có chức năng kết nối các thiết bị IoT này với nền tảng IoT.

2. Inlets (Cổng vào) chứa một nhóm các dịch vụ AWS như [**AWS IoT Core**](https://aws.amazon.com/iot-core/) và [**Amazon API Gateway**](https://aws.amazon.com/api-gateway/) để thu thập các tín hiệu phát hiện từ IoT thông qua giao thức MQTTS hoặc HTTPS và phân phối chúng đến luồng dữ liệu nguồn bằng [**Amazon Kinesis Data Streams**](https://aws.amazon.com/kinesis/data-streams).

3. Ứng dụng tổng hợp (aggregation application) tiến hành lọc các tín hiệu IoT, tổng hợp chúng trong một khoảng thời gian cố định và đẩy dữ liệu đã tổng hợp vào luồng dữ liệu đích.

4. Trình tạo sự kiện (Event producers) là sự kết hợp của các dịch vụ có trạng thái khác nhau nhằm tạo ra các sự kiện IoT như geofencing, tính sẵn sàng, bottom state (trạng thái cơ bản), và in-transit (đang vận chuyển).

5. Outlets (Cổng ra), bao gồm các dịch vụ như [**Amazon EventBridge**](https://aws.amazon.com/eventbridge/), [**Amazon Data Firehose**](https://aws.amazon.com/firehose/), và Kinesis Data Streams, phân phối các sự kiện đã tạo ra tới đối tượng tiêu thụ (consumers).

6. Consumers, tức là các nhóm nội bộ, sẽ diễn giải các sự kiện IoT và xây dựng logic nghiệp vụ dựa trên chúng.

Thành phần cốt lõi của kiến trúc này là ứng dụng tổng hợp. Thành phần này ban đầu được triển khai bằng một công nghệ xử lý luồng cũ. Vì một số lý do mà chúng tôi sẽ thảo luận ngắn gọn ngay sau đây, PostNL đã quyết định nâng cấp thành phần trọng yếu này. Hành trình thay thế hệ thống xử lý luồng cũ bằng Managed Service cho Apache Flink chính là trọng tâm của phần còn lại trong bài viết này.

## **Quyết định di chuyển ứng dụng tổng hợp sang Managed Service cho Apache Flink**
Khi số lượng thiết bị được kết nối ngày một tăng lên, nhu cầu về một nền tảng mạnh mẽ và có khả năng mở rộng để xử lý, tổng hợp khối lượng dữ liệu IoT khổng lồ cũng tăng theo. Sau khi phân tích kỹ lưỡng, PostNL đã chọn di chuyển sang Managed Service cho Apache Flink, xuất phát từ một số cân nhắc chiến lược phù hợp với nhu cầu kinh doanh đang thay đổi:

- **Cải tiến việc tổng hợp dữ liệu** – Sử dụng các khả năng mạnh mẽ của Apache Flink trong xử lý dữ liệu thời gian thực giúp PostNL tổng hợp hiệu quả dữ liệu IoT thô từ các nguồn khác nhau. Khả năng mở rộng logic tổng hợp vượt ra ngoài những gì giải pháp hiện tại cung cấp có thể mở ra những phân tích tinh vi hơn và quy trình ra quyết định sáng suốt hơn.

- **Khả năng mở rộng** – Dịch vụ được quản lý này cung cấp khả năng mở rộng ứng dụng theo chiều ngang. Điều này cho phép PostNL xử lý khối lượng dữ liệu ngày càng tăng một cách dễ dàng khi số lượng thiết bị IoT tăng lên. Khả năng mở rộng này đồng nghĩa với việc năng lực xử lý dữ liệu có thể theo kịp sự tăng trưởng của doanh nghiệp.

- **Tập trung vào mảng kinh doanh cốt lõi** – Bằng cách áp dụng một dịch vụ được quản lý, nhóm phát triển nền tảng IoT có thể tập trung vào việc triển khai logic nghiệp vụ và phát triển các trường hợp sử dụng mới. Quá trình làm quen (learning curve) cũng như gánh nặng vận hành Apache Flink ở quy mô lớn nếu có sẽ làm phân tán nguồn năng lượng và tài nguyên quý giá của một nhóm tương đối nhỏ, làm chậm lại quá trình ứng dụng.

- **Hiệu quả về chi phí** – Managed Service cho Apache Flink áp dụng mô hình thanh toán theo mức sử dụng (pay-as-you-go) phù hợp với ngân sách vận hành. Sự linh hoạt này đặc biệt có lợi trong việc quản lý chi phí nhằm đồng điệu với các nhu cầu xử lý dữ liệu luôn biến động.

**Những thách thức trong việc xử lý sự kiện trễ (late events)**

Các trường hợp sử dụng xử lý luồng thông thường yêu cầu tổng hợp các sự kiện dựa trên thời điểm chúng được tạo ra. Khái niệm này gọi là ngữ nghĩa thời gian sự kiện (event time semantics). Khi triển khai loại logic này, bạn có thể gặp phải vấn đề với các sự kiện bị chậm trễ, trong đó sự kiện được truyền tới hệ thống xử lý của bạn quá trễ, rất lâu sau các sự kiện khác được tạo ra trong cùng một khoảng thời gian.

Các sự kiện trễ diễn ra phổ biến trong mảng IoT vì những lý do cố hữu của môi trường, chẳng hạn như độ trễ mạng, lỗi thiết bị, ngắt kết nối tạm thời, hoặc thời gian chết (downtime). Thiết bị IoT thường giao tiếp qua mạng không dây, điều này có thể dẫn đến sự chậm trễ trong việc truyền các gói dữ liệu. Và đôi khi chúng có thể gặp sự cố kết nối đứt quãng, khiến cho dữ liệu được lưu vào bộ đệm (buffer) và gửi theo một khối sau khi kết nối được khôi phục. Điều này dẫn đến việc các sự kiện được xử lý không theo thứ tự—một số sự kiện có thể được xử lý chậm hơn vài phút so với các sự kiện khác vốn được tạo ra cùng một thời điểm.

Hãy tưởng tượng bạn muốn tổng hợp các sự kiện do các thiết bị tạo ra trong một khoảng cửa sổ 10 giây cụ thể. Nếu các sự kiện có thể bị trễ vài phút, làm sao bạn chắc chắn rằng mình đã nhận được toàn bộ các sự kiện được tạo ra trong 10 giây đó?

Một cách triển khai đơn giản là chỉ cần đợi thêm vài phút để các sự kiện trễ đến nơi. Nhưng phương pháp này đồng nghĩa với việc bạn không thể tính toán kết quả của quá trình tổng hợp cho đến vài phút sau đó, làm tăng độ trễ đầu ra (output latency). Một giải pháp khác là chờ trong vài giây rồi vứt bỏ bất kỳ sự kiện nào đến muộn hơn.

Việc tăng độ trễ hoặc loại bỏ các sự kiện có thể chứa thông tin quan trọng đều không phải là lựa chọn mong muốn của doanh nghiệp. Giải pháp cần phải là một sự thỏa hiệp hợp lý, một sự đánh đổi khéo léo giữa độ trễ và tính toàn vẹn (completeness).

Apache Flink hỗ trợ sẵn (out of the box) ngữ nghĩa thời gian sự kiện. Trái ngược với các framework xử lý luồng khác, Flink cung cấp nhiều tùy chọn để giải quyết các sự kiện bị trễ. Chúng ta sẽ cùng đi sâu vào cách Apache Flink xử lý các sự kiện đến muộn ở phần tiếp theo.

**Một API xử lý luồng mạnh mẽ**

Apache Flink cung cấp một tập hợp phong phú các toán tử (operators) và thư viện cho những tác vụ xử lý dữ liệu phổ biến, bao gồm tạo khung cửa sổ (windowing), phép nối (joins), bộ lọc (filters) và các phép biến đổi (transformations). Nó cũng bao gồm hơn 40 trình kết nối (connectors) tới nhiều nguồn và đích dữ liệu khác nhau, từ các hệ thống truyền phát (streaming) như [**Apache Kafka**](https://kafka.apache.org/) và [**Amazon Managed Streaming cho Apache Kafka**](https://aws.amazon.com/msk/), hoặc Kinesis Data Streams, cho đến cơ sở dữ liệu, các hệ thống tệp tin và kho lưu trữ đối tượng như [**Amazon Simple Storage Service**](https://aws.amazon.com/s3/) (Amazon S3).

Nhưng đặc điểm quan trọng nhất đối với PostNL là việc Apache Flink cung cấp các [**API với những cấp độ trừu tượng khác nhau**](https://nightlies.apache.org/flink/flink-docs-master/docs/concepts/overview/#flinks-apis). Bạn có thể bắt đầu với một cấp độ trừu tượng cao hơn, chẳng hạn như [**SQL**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/table/sql/overview/), hoặc [**Table API**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/table/overview/). Các API này trừu tượng hóa dữ liệu luồng dưới dạng các bảng (tables) quen thuộc hơn, giúp người dùng dễ dàng nắm bắt hơn trong các trường hợp sử dụng đơn giản. Nếu logic của bạn trở nên phức tạp hơn, bạn có thể chuyển sang cấp độ trừu tượng thấp hơn là [**DataStream API**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/overview/), nơi các luồng được biểu diễn một cách tự nhiên (natively), gần sát hơn với quá trình xử lý đang diễn ra bên trong Apache Flink. Nếu bạn cần mức độ kiểm soát cực kỳ chi tiết (fine-grained) về cách thức từng sự kiện riêng lẻ được xử lý ra sao, bạn có thể chuyển hẳn sang [**Process Function**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/process_function/).

Một bài học quan trọng là việc lựa chọn một cấp độ trừu tượng cho ứng dụng của bạn không phải là một quyết định kiến trúc bất biến không thể đảo ngược. Trong cùng một ứng dụng, bạn hoàn toàn có thể kết hợp nhiều API khác nhau, tùy thuộc vào mức độ kiểm soát bạn cần tại một bước thao tác cụ thể.

**Mở rộng quy mô theo chiều ngang**

Để xử lý hàng tỷ sự kiện thô và phát triển song hành với doanh nghiệp, khả năng mở rộng là một yêu cầu tất yếu đối với PostNL. Apache Flink được thiết kế để mở rộng theo chiều ngang, phân tán quá trình xử lý cũng như trạng thái ứng dụng trên nhiều nút (nodes) khác nhau, và sở hữu khả năng tiếp tục tăng quy mô hơn nữa khi khối lượng công việc phát triển.

Đối với trường hợp sử dụng cụ thể này, PostNL phải tổng hợp khối lượng khổng lồ các sự kiện thô mang các đặc điểm tương đồng và phân theo thời gian, để giảm bớt số lượng phần tử (cardinality) và làm cho luồng dữ liệu có thể dễ dàng quản lý đối với những hệ thống hạ nguồn. Các hoạt động tổng hợp này vượt xa các phép chuyển đổi đơn giản nơi chỉ xử lý từng sự kiện một cách đơn lẻ. Chúng đòi hỏi một framework có khả năng xử lý luồng có lưu trạng thái (stateful stream processing). Đây chính xác là kiểu ứng dụng mà Apache Flink được sinh ra để giải quyết.

**Ngữ nghĩa thời gian sự kiện nâng cao**

Apache Flink đặc biệt nhấn mạnh vào xử lý theo thời gian sự kiện (event time processing), cho phép xử lý dữ liệu một cách nhất quán và cực kỳ chuẩn xác căn cứ vào thời điểm mà dữ liệu đã thực sự xảy ra. Thông qua việc hỗ trợ tích hợp sẵn ngữ nghĩa thời gian sự kiện, Flink có thể xử lý trơn tru các sự kiện lộn xộn không đúng trình tự cũng như dữ liệu trễ muộn một cách khéo léo. Khả năng này mang ý nghĩa cốt lõi đối với PostNL. Như đã đề cập, các sự kiện do IoT tạo ra có thể sẽ đến chậm và sai thứ tự. Tuy nhiên, logic tổng hợp bắt buộc phải dựa trên chính xác khoảnh khắc mà đo lường thực sự được thực hiện bởi thiết bị — tức là thời gian sự kiện (event time) — chứ không phải lúc nó được hệ thống xử lý (processing time).

**Độ bền bỉ và các cam kết (Guarantees)**

PostNL phải đảm bảo rằng không một dữ liệu nào gửi từ thiết bị bị mất, ngay cả khi xảy ra sự cố hay ứng dụng phải khởi động lại. Apache Flink cung cấp cam kết về khả năng chịu lỗi cực kỳ mạnh mẽ thông qua cơ chế lưu trạng thái trạm (checkpointing) phân tán dựa trên các bản chụp nhanh (snapshot). Khi có sự cố, Flink có thể khôi phục trạng thái của các phép tính và duy trì được ngữ nghĩa chính xác một lần (exactly-once semantics) của kết quả. Ví dụ, mỗi sự kiện từ một thiết bị sẽ không bao giờ bị bỏ sót cũng như không bao giờ bị tính trùng hai lần, kể cả trong những trường hợp ứng dụng gặp trục trặc.

## **Hành trình lựa chọn API Apache Flink phù hợp**
Một yêu cầu mang tính thiết yếu của lần di chuyển này là phải mô phỏng lại chính xác hành vi của ứng dụng tổng hợp cũ trước kia, theo đúng kỳ vọng của các hệ thống hạ nguồn do chúng không thể bị thay đổi. Điều này đã kéo theo không ít những thách thức, đặc biệt xoay quanh ngữ nghĩa phân khung cửa sổ thời gian (windowing) và quá trình xử lý sự kiện trễ.

Như chúng ta đã thấy, ở lĩnh vực IoT, các sự kiện có thể trượt khỏi trình tự gốc chênh lệch lên tới vài phút. Apache Flink cung cấp hai khái niệm bậc cao để triển khai ngữ nghĩa thời gian sự kiện với các sự kiện lệch trình tự: [**watermarks (dấu thời gian chuẩn)**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/concepts/time/#event-time-and-watermarks) và [**allowed lateness (độ trễ cho phép)**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/windows/#allowed-lateness).

Apache Flink cũng mang đến một loạt các API rất linh hoạt sở hữu các mức độ trừu tượng khác nhau. Sau một thời gian phân tích ban đầu, [**Flink-SQL và Table API**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/table/overview/) đã bị loại bỏ. Các mức độ trừu tượng cao hơn này có cung cấp tính năng phân cửa sổ nâng cao và các ngữ nghĩa thời gian sự kiện, tuy nhiên lại không thể đưa ra khả năng kiểm soát thật sự chi tiết (fine-grained control) mà PostNL cần nhằm sao chép lại một cách hoàn hảo cơ chế hành vi của ứng dụng cũ.

Mức trừu tượng thấp hơn là [**DataStream API**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/overview/) cũng hỗ trợ những tính năng tổng hợp tạo cửa sổ thời gian, đồng thời cho phép bạn tùy chỉnh thêm cách thức hoạt động với các [**triggers (trình kích hoạt)**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/windows/#triggers), [**evictors (trình loại bỏ)**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/windows/#evictors) tùy chỉnh, và xử lý các sự kiện trễ thông qua việc thiết lập một thông số [**allowed lateness**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/windows/#allowed-lateness).

Thật không may, ứng dụng cũ (legacy) lại được thiết kế để xử lý các sự kiện trễ theo một phong cách vô cùng kỳ lạ. Hậu quả là hình thành một chuỗi logic kết hợp lai tạp giữa thời gian sự kiện và thời gian xử lý vốn dĩ không thể nào được tái tạo lại dễ dàng thông qua các nguyên hàm (primitives) Flink cấp cao.

May mắn thay, Apache Flink cung cấp một mức độ trừu tượng thấp hơn nữa, đó là [**ProcessFunction API**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/process_function/#the-processfunction). Dưới API này, bạn có được toàn quyền kiểm soát tinh chỉnh nhất đối với trạng thái của ứng dụng và có thể sử dụng [**timers (bộ đếm thời gian)**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/process_function/#timers) để triển khai thực tế hầu như mọi loại logic tùy chỉnh dựa trên mốc thời gian.

PostNL đã quyết định đi theo hướng này. Việc tổng hợp được lập trình bằng một [**KeyedProcessFunction**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/process_function/#the-keyedprocessfunction) chuyên cung cấp phương pháp để tiến hành bất kỳ việc xử lý có trạng thái nào trên các luồng có khóa (keyed streams)—tức là các luồng phân vùng logic. Các sự kiện thô xuất phát từ mỗi thiết bị IoT sẽ được tổng hợp lại tính theo thời gian sự kiện (event time - mốc thời gian do thiết bị gốc ghi lại trên sự kiện) và đầu ra của mỗi cửa sổ thời gian sẽ được phát ra tính theo thời gian xử lý (processing time - thời gian hệ thống hiện tại).

Kiểm soát chi tiết này rốt cuộc đã giúp cho PostNL mô phỏng lại chuẩn xác hoàn toàn cách thức hoạt động mà các ứng dụng hạ nguồn hằng mong đợi.

## **Hành trình sẵn sàng để đưa vào sản xuất (production readiness)**
Chúng ta hãy cùng khám phá toàn bộ chặng đường chuyển dịch sang hệ thống Managed Service cho Apache Flink, bắt đầu từ điểm khởi chạy dự án cho tới bước triển khai thực tế đưa vào sản xuất.

**Xác định các yêu cầu**

Bước đầu tiên trong toàn bộ quá trình di chuyển được tập trung vào việc nghiên cứu hiểu rõ chi tiết từ trong ra ngoài kiến trúc của hệ thống hiện hữu và các chỉ số hiệu năng hoạt động. Mục đích nhằm cung cấp một quá trình chuyển đổi mượt mà và liền mạch sang hệ thống Managed Service cho Apache Flink với độ ảnh hưởng thấp nhất đến các hoạt động đang trong tiến trình xử lý.

**Tìm hiểu về Apache Flink**

PostNL cần tự tạo ra sự quen thuộc với ứng dụng Managed Service cho Apache Flink cũng như các tính năng xử lý dòng luồng của nó, trong đó bao hàm luôn cả các [**chiến lược phân khung (windowing)**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/windows/) tích hợp sẵn, những hàm tổng hợp (aggregation functions), sự khác biệt căn bản giữa [**thời gian sự kiện (event time) và thời gian xử lý (processing time)**](https://nightlies.apache.org/flink/flink-docs-release-1.18/docs/concepts/time/#notions-of-time-event-time-and-processing-time), và cuối cùng là [**KeyProcessFunction**](https://nightlies.apache.org/flink/flink-docs-release-1.19/docs/dev/datastream/operators/process_function/#the-keyedprocessfunction) đi cùng những cơ chế dùng xử lý các sự kiện trễ.

Nhiều lựa chọn thay thế khác nhau đã được cân nhắc cẩn thận, thông qua việc sử dụng các tính năng nền (primitives) có sẵn ra lò (out of the box) của Apache Flink cho cả logic thời gian sự kiện cũng như các sự kiện chậm trễ. Yêu cầu khắt khe nhất chính là việc bắt buộc tái sinh lại chính xác y hệt hành vi hệ thống kế thừa (legacy) cũ. Khả năng được chuyển hướng xuống khai thác cấp độ trừu tượng thấp hơn đã mang lại hiệu quả to lớn. Với khả năng điều khiển tinh chỉnh sắc nét nhất mà ProcessFunction API mang lại, PostNL đã hoàn thành nhiệm vụ quản lý thành công các sự kiện trễ giống y như một bản sao hoàn hảo của ứng dụng cũ.

**Thiết kế và triển khai ProcessFunction**

Logic nghiệp vụ này được tạo hình thông qua ProcessFunction để làm giả mạo cơ chế đặc thù của ứng dụng cũ trong phương pháp xử lý những sự kiện trễ nhưng không được phép làm chậm kết quả trả về ban đầu quá tay. PostNL đi đến quyết định dùng ngôn ngữ lập trình Java để code triển khai, do Java vốn là ngôn ngữ bản địa ưu tiên của Apache Flink. Hơn nữa, Apache Flink cho phép bạn phát triển và kiểm thử sản phẩm trực tiếp từ máy cục bộ cá nhân (locally), ở ngay trong môi trường phát triển tích hợp (IDE) quen tay nhất của bạn, thoải mái dùng toàn bộ mọi công cụ dò lỗi (debug) hiện hành, trước cả khi tải triển khai thật lên Managed Service cho Apache Flink. Đội ngũ đã sử dụng Java 11 cùng trình biên dịch Maven để làm công cụ thực thi. Để tìm hiểu chuyên sâu hơn về hệ quy chiếu yêu cầu IDE, có thể xem bài [**Bắt đầu với Amazon Managed Service cho Apache Flink (DataStream API)**](https://docs.aws.amazon.com/managed-flink/latest/java/getting-started.html).

**Kiểm thử và xác thực**

Biểu đồ sau mô phỏng toàn bộ kiến trúc được sử dụng trong việc xác minh chạy thử nghiệm ứng dụng mới.

## ![](/images/3-BlogsTranslated/3.1-Blog1/image_3.jpg)

Để làm bằng chứng xác thực hoạt động xử lý từ ProcessFunction kèm các cơ chế quản lý sự kiện trễ, nhiều bài kiểm thử tích hợp (integration tests) đã được phác thảo để có thể chạy đồng thời cho cả ứng dụng cũ (legacy) lẫn ứng dụng Managed Service cho Flink theo kiểu song hành (Bước 3 và 4). Quá trình vận hành song song này giúp PostNL được trực tiếp phân tích so sánh đối chiếu số liệu xuất ra giữa từng ứng dụng đặt dưới những điều kiện hệ thống giống nhau như đúc. Rất nhiều kịch bản bài kiểm thử tích hợp tự động bơm dữ liệu đầu vào chạy vào các luồng nguồn (2) song song cùng nhau (7) và theo dõi chờ tới chừng nào cả cửa sổ tổng hợp đó hoàn thành khép lại, mới kéo lượng số liệu tổng hợp được từ luồng đích ra để so kè (8). Hàng loạt những bài test tích hợp này luôn được hệ thống đường ống tự động CI/CD kích nổ chạy ngay sau khi quá trình triển khai cấu trúc hạ tầng cơ sở kết thúc hoàn hảo. Trong lúc thực hiện chạy kiểm thử tích hợp, sự tập trung lớn nhất đặt vào trọng tâm phải đạt mức đồng nhất kết quả dữ liệu hoàn hảo cùng độ chuẩn xác về năng lực tính toán giữa ứng dụng cũ và hệ thống mới từ Managed Service cho Flink. Các đầu ra dòng chảy, cụm dữ liệu tổng hợp, cũng như các điểm độ trễ khi xử lý (latencies) đều được đem đo lường cẩn thận nhằm khẳng định rằng quá trình chuyển đổi này tuyệt nhiên không gây phát sinh nên bất kì hố sai lệch ngoài mong đợi nào cả. Đóng vai trò làm bệ đỡ xây dựng viết kịch bản chạy thử tích hợp, [**Robot Framework**](https://robotframework.org/), một nền tảng tạo kịch bản tự động hóa mã nguồn mở, đã được ưu ái sử dụng.

Khi loạt kiểm thử tích hợp báo xanh thành công, vẫn còn cần vượt qua thêm một lớp phòng vệ rà soát nữa: kiểm thử đầu cuối (end-to-end tests). Gần như y chang với các bộ bài tập kiểm thử tích hợp, mảng end-to-end tests sẽ do CI/CD pipeline động kích hoạt sau khi hạ tầng nền tảng hoàn thiện xong xuôi. Ở giai đoạn này, nhiều bộ test-case end-to-end trực tiếp đẩy số liệu vào AWS IoT Core (1) thông qua chạy song song nhịp nhàng (9) để lấy mẫu check chéo kết quả tổng hợp được xuất xưởng chuyển lưu trong đích lưu trữ bucket S3 (5, 6) được chiết ra trực tiếp từ luồng đầu ra rồi mới chốt hạ việc so sánh đo kiểm (10).

**Triển khai**

PostNL đã lựa chọn cách chạy ứng dụng Flink đời mới dưới dạng chế độ ẩn song song (shadow mode). Hệ thống ứng dụng mới chạy trôi chảy qua một khoảng thời gian sánh vai đi chung cạnh ứng dụng cũ, cùng “ăn” vào chính xác chung một đầu vào thông tin (inputs) đó, rồi xả ra dữ liệu kết xuất từ cả hai ứng dụng ném vào hệ thống hồ dữ liệu (data lake) đặt trên Amazon S3. Chiến thuật này vừa tạo cơ hội vàng mang các kết quả của hai nền tảng ra mổ xẻ đánh giá ngay lúc dùng dữ liệu thật đang sống ngoài production, lại đồng thời làm bài test thực lực vững chãi kiểm định tính ổn định và sức chịu đựng hiệu năng làm việc cho ứng dụng tân tiến.

**Tối ưu hóa hiệu năng**

Trong kỳ nâng cấp di dời này, toàn bộ thành viên ban IoT của PostNL được hấp thu một khối lượng hiểu biết dồi dào về kỹ thuật tinh chỉnh (fine-tuned) hiệu năng cho Flink nhằm chạm tay tới đỉnh cao xử lý, bao quát trên mọi mặt từ số lượng lượng dữ liệu nạp vào, tốc độ guồng máy làm việc, cho đến phương án quản lý chuyên sâu hiệu quả đám dữ liệu đến muộn (late event). Một trong những mặt thú vị đặc biệt đó là công việc phân tích kiểm toán lượng kích thước trạng thái (state size) để củng cố tin cậy rằng nó sẽ không phồng to mất kiểm soát vô tận theo đường dài (unbounded). Một sự rủi ro cực kì dễ mắc bẫy nếu áp dụng cơ chế tự điểu khiển linh hoạt độ hạt sâu bằng ProcessFunction đó là trạng thái bị rò rỉ (state leak). Hiện tượng này xảy ra mỗi lúc quá trình tự lập trình code can thiệp thô bạo thẳng vào State bên trong khu vực ProcessFunction làm đánh rơi mất vô tình vài ca góc khuất (corner cases) khiến trạng thái lưu không tài nào bị hệ thống ra lệnh xóa dọn đi được nữa. Bỏ quên chúng sẽ vô tình cổ xúy State giãn nở lớn lên không hồi kết. Bởi vì ứng dụng streaming mang thân phận là hệ thống sẽ phải chạy liên tu bất tận 24/7, trạng thái hệ thống phì đại béo mập quá mức sẽ đánh quỵ hiệu năng ứng dụng xuống dốc và về lâu dài làm chết khô khoáng cạn kiệt bộ nhớ hoặc dung lượng lưu trữ cục bộ ổ cứng.

Thông qua học hỏi ở đợt thử lửa này, PostNL thành công lèo lái đến mức độ giao thoa hài hòa tỷ lệ cực tốt trong việc cho phép ứng dụng chạy song song (parallelism) phân bổ xen kẽ giữa các nguồn năng lực cấu hình thiết bị — gồm vi xử lý (compute), vùng nhớ (memory), hay không gian giữ liệu (storage) — nhằm xử lý trơn tru thông mượt hàng tấn tác vụ trong ngày mà giảm thiểu cực kì biểu hiện ùn ứ giật lag, đối phó uyển chuyển gọn nhẹ đợt bùng nổ điểm (peaks) mà hiếm khi phải dư xài hao hụt tài nguyên bừa bãi (over-provisioning), đẩy lùi triệt để giới hạn kết hợp cả về hiệu suất lao động cùng độ hiệu quả về mặt ngân sách (cost-effectiveness).

**Chuyển đổi cuối cùng**

Sau khi cho ứng dụng mới chạy chế độ song song ẩn (shadow mode) suốt khoảng thời gian định trước, đội dự án đã tự tin tuyên bố ứng dụng thực sự vững bền (stable) kèm theo chất lượng thông số đẩy ra (output) chuẩn như chỉ tiêu đề ra lúc đầu. Nền tảng lõi IoT trực thuộc PostNL vào ngày lành cuối cùng cũng ấn nút sang số đổi đường dứt điểm toàn bộ tiến vào khâu sản xuất thật (production) rồi từ tốn thực hiện nghi thức hạ đài đánh sập ứng dụng hệ di sản cũ kĩ (legacy application).

## **Những bài học quan trọng**
Từ hàng sa số bài học thu nhận lúc tham gia chuyến hành hương thỉnh chân kinh sử dụng Managed Service cho Apache Flink, một lượng nhỏ bí quyết được định hình vô cùng then chốt, minh chứng qua vai trò át chủ bài lúc đội nhóm quyết tâm bung tỏa ra thâu tóm thêm một loạt các nhóm mô hình công việc đa diện mới:

- **Hiểu về ngữ nghĩa thời gian sự kiện** – Sự thẩm thấu một cách thấu đáo kiến thức quanh ngữ nghĩa thời gian sự kiện nắm giữ vận mệnh quyết định ở sâu trong bộ não Apache Flink giúp cho ta lập trình thành công hệ hoạt động phụ thuộc nhiều yếu tố thời gian cực kì xác thực. Kiến thức tinh hoa này duy trì cho hàng loạt sự kiện đều có thứ tự xếp hàng được bảo toàn y hệt thứ tự thời gian lịch sử thật của nó khi xảy ra tại hiện trường.

- **Sử dụng API hùng mạnh của Apache Flink** – Bức tường thành API của nhà Apache Flink cấp phát sinh lực tái tạo ra những công trình ứng dụng streaming lưu giữ trạng thái siêu tối tân tinh xảo nằm bỏ xa năng lực gói gọn chỉ xử lý cửa sổ đơn lẻ hoặc lấy tổng gom dữ liệu bình thường. Nhu cầu hấp thụ bao quát tổng lực những thế võ uy lực mà kho tàng API này giăng sẵn vốn được xem là công cụ trọng đạo để đương đầu chinh phục những ca giải bài toán xử lý thông tin phức tạp (sophisticated data processing challenges).

- **Sức mạnh càng lớn, trách nhiệm càng cao** – Chức năng vượt bậc đi song đôi thuộc mảng API Apache Flink gánh vác kèm theo một độ chịu trách nhiệm khổng lồ vĩ đại. Nhóm làm kỹ sư phần mềm bắt buộc phải gánh trách nhiệm kiểm duyệt đảm bảo phần mềm mã code phải đạt độ hiệu quả vận hành tối thượng, dễ chẩn đoán chăm sóc sửa chữa bảo trì, cũng như không hề bị rung lắc sụp đổ, điều đó thúc ép người làm quản trị cực kì khéo tay khi điều hướng sắp xếp tài nguyên cùng thái độ làm việc giữ tính kỷ luật làm chuẩn các quy định viết code và dựng hình thiết kế cấu trúc tốt nhất.

- **Không trộn lẫn logic thời gian sự kiện và thời gian xử lý** – Thao tác kết hợp nhào trộn chung hai khái niệm Thời gian sự kiện và Thời gian quy trình vào cùng mâm để vò tổng hợp số liệu dữ liệu ẩn tàng hiểm họa tự khơi dậy nhiều nút thắt thử thách vô tiền khoáng hậu khác lạ. Nó tước đoạt thẳng thừng quyền năng bạn được sử dụng ké những cụm chức năng đẳng cấp cao mà vốn dĩ Apache Flink đã bỏ sẵn vô trong hộp quà đồ nghề out of the box của nó. Việc đào quá sâu đụng độ chạm đến nền móng tầng hạ đẳng nhất trong tháp trừu tượng (lowest level of abstractions) mảng hệ sinh thái API của Apache Flink vốn sẽ cung cấp đủ dụng cụ chế tác nên bất kì đường lối tính toán bằng mốc thời gian riêng biệt nào (custom time-based logic), tuy nhiên cần phải bắt chủ nhân đầu tư thiết kế tỉ mỉ cẩn trọng cao độ để đảm bảo sự đo lường là chính xác đi chung với thành quả đáp số kịp thời, kèm với quy trình phải test kiểm đi xét lại thật nghiêm túc sâu rộng đảm nhận vai trò rào chắn phòng vệ hiệu năng.

## **Kết luận**
Trong chặng đường tiếp thu ứng dụng Apache Flink, ban nòng cốt PostNL đã thấu hiểu cách để hệ thống API xuất sắc của Apache Flink giúp người xài lập trình tạo nên logic quy trình hoạt động doanh nghiệp phức tạp nhất. Ban đội ngũ nhóm này đã bắt đầu cảm thấy sự khâm phục tôn trọng với phương thức mà Apache Flink dùng đòn bẩy tháo gỡ nhiều rào cản bài toán sự cố hỗn tạp khác nhau, giờ đây họ đã lên hẳn chương trình phác thảo bản kế hoạch bành trướng nó áp dụng trực tiếp cho lượng lớn quy mô khổng lồ bài toán xử lý luồng streaming mới đa dạng.

Cùng sự trợ lực mạnh mẽ từ Managed Service cho Apache Flink, cả team toàn quyền nhắm mắt thoải mái buông bỏ lo âu vào việc gầy dựng giá trị tinh hoa cho doanh nghiệp và thiết lập mô hình logic cần thiết, tuyệt nhiên hất văng sự gánh nặng phiền hà lo toan cài đặt hoặc chăm lo gánh vác việc nuôi dưỡng giữ gìn cả một cụm server máy chủ Apache Flink.

Nếu muốn tìm hiểu chuyên sâu sắc thêm về nền tảng Managed Service cho Apache Flink cùng bí kíp lựa chọn tìm ra lựa chọn tính năng quản lý với API bám sát bài toán (use case) của riêng mình, hãy thử nhấp đọc tham khảo thông tin ở [**Amazon Managed Service cho Apache Flink là gì**](https://docs.aws.amazon.com/managed-flink/latest/java/what-is.html). Kế đó nếu muốn tận tay tự mình thử nghiệm nhúng tay lập trình phát triển (develop), phóng triển khai (deploy), hay điều khiển lái vận hành (operate) cụm công cụ Flink apps đặt tại AWS, vui lòng truy cập nghiên cứu tại mục [**Hội thảo Amazon Managed Service cho Apache Flink**](https://catalog.workshops.aws/managed-flink/en-US).

**Bài đăng trên AWS Study Group**
## ![](/images/image_1.png)

## **Về các tác giả**

<div style="display:flex; flex-direction:column; gap:1rem;">

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.1-Blog1/image_4.jpg" alt="Çağrı Çakır" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Çağrı Çakır</strong><br/>
      Çağrı Çakır là Kỹ sư Phần mềm Trưởng cho nền tảng IoT của PostNL, nơi ông quản lý kiến trúc xử lý hàng tỷ sự kiện mỗi ngày. Là một Kiến trúc Sư Giải pháp Chuyên nghiệp được Chứng nhận của AWS (AWS Certified Solutions Architect Professional), ông chuyên về thiết kế và triển khai các kiến trúc hướng sự kiện (event-driven architectures) và các giải pháp xử lý luồng trên quy mô lớn. Ông đam mê khai thác sức mạnh của dữ liệu thời gian thực và tận tâm hướng tới việc tối ưu hóa hiệu quả hoạt động cũng như đổi mới các hệ thống có khả năng mở rộng.
    </div>
  </div>

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.1-Blog1/image_5.jpg" alt="Özge Kavalcı" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Özge Kavalcı</strong><br/>
      Özge Kavalcı làm việc với vai trò là Kỹ sư Giải pháp Cấp cao cho nền tảng IoT của PostNL và có niềm đam mê xây dựng các giải pháp tiên tiến tích hợp với bối cảnh IoT. Với tư cách là Kiến trúc Sư Giải pháp được Chứng nhận của AWS, cô chuyên thiết kế và triển khai các kiến trúc serverless (phi máy chủ) có khả năng mở rộng cao và các giải pháp xử lý luồng thời gian thực có thể giải quyết được các khối lượng công việc khó lường. Nhằm mở khóa toàn bộ tiềm năng của dữ liệu thời gian thực, cô dành trọn tâm huyết để định hình tương lai của quá trình tích hợp IoT.
    </div>
  </div>

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.1-Blog1/image_6.jpg" alt="Amit Singh" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Amit Singh</strong><br/>
      Amit Singh làm việc với vai trò là Kiến trúc Sư Giải pháp Cấp cao tại AWS, đồng hành cùng các khách hàng doanh nghiệp về đề xuất giá trị của AWS và tham gia vào các cuộc thảo luận kiến trúc chuyên sâu nhằm đảm bảo các giải pháp được thiết kế thành công để triển khai trên đám mây. Công việc này bao gồm cả việc xây dựng các mối quan hệ sâu sắc với các chuyên gia kỹ thuật cấp cao (senior technical individuals) để hỗ trợ họ trở thành những người ủng hộ đám mây thực thụ. Trong thời gian rảnh rỗi, anh thích dành thời gian cho gia đình và tìm hiểu thêm về mọi thứ liên quan đến đám mây.
    </div>
  </div>

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.1-Blog1/image_7.jpg" alt="Lorenzo Nicora" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Lorenzo Nicora</strong><br/>
      Lorenzo Nicora là Kiến trúc Sư Giải pháp Xử lý Luồng Cấp cao (Senior Streaming Solutions Architect) tại AWS chuyên hỗ trợ cho các khách hàng trên toàn khu vực EMEA. Anh đã có nhiều năm cống hiến để xây dựng các hệ thống chuyên sâu về dữ liệu, lấy đám mây làm trung tâm, từng làm việc trong lĩnh vực tài chính thông qua cả các công ty tư vấn cũng như các công ty công nghệ tài chính (fintech). Anh đã sử dụng sâu rộng nhiều loại công nghệ mã nguồn mở khác nhau và từng đóng góp cho một số dự án, trong đó có cả dự án Apache Flink.
    </div>
  </div>
  
</div>