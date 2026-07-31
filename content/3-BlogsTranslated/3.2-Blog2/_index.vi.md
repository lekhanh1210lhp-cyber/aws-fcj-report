---
title: "Blog 2"
date: "2025-07-25"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# **Tối ưu hóa phân tích dữ liệu IoT công nghiệp với Amazon Data Firehose và Amazon S3 Tables cùng Apache Iceberg**

_bởi Ashok Padmanabhan, Joyson Neville Lewis, và Anil Vure vào ngày 25 THÁNG 7 NĂM 2025 trong Amazon Data Firehose, Amazon S3 Tables, Analytics, AWS IoT Greengrass, Internet of Things, Technical How-to_

Các tổ chức sản xuất đang chạy đua để số hóa hoạt động của họ thông qua các sáng kiến Công nghiệp 4.0. Một thách thức chính mà họ phải đối mặt là việc thu thập, xử lý và phân tích dữ liệu thời gian thực từ các thiết bị công nghiệp để cho phép ra quyết định dựa trên dữ liệu.

Các cơ sở sản xuất hiện đại tạo ra lượng dữ liệu thời gian thực khổng lồ từ các dây chuyền sản xuất của họ. Việc thu thập dữ liệu giá trị này đòi hỏi một kiến trúc hai tầng: đầu tiên, một thiết bị biên (edge device) hiểu các giao thức công nghiệp sẽ thu thập dữ liệu trực tiếp từ các cảm biến trên sàn nhà máy. Sau đó, các cổng biên (edge gateway) này sẽ đệm (buffer) và truyền dữ liệu một cách an toàn đến AWS Cloud, đảm bảo độ tin cậy trong các trường hợp gián đoạn mạng.

Trong bài viết này, chúng tôi sẽ trình bày cách sử dụng các tích hợp dịch vụ của AWS để giảm thiểu mã tùy chỉnh (custom code) trong khi vẫn cung cấp một nền tảng mạnh mẽ cho việc thu thập, xử lý và phân tích dữ liệu công nghiệp. Bằng cách sử dụng Amazon S3 Tables và các tối ưu hóa được tích hợp sẵn của nó, bạn có thể tối đa hóa hiệu suất truy vấn và giảm thiểu chi phí mà không cần thiết lập thêm cơ sở hạ tầng. Ngoài ra, AWS IoT Greengrass hỗ trợ VPC endpoints, giúp bạn có thể giao tiếp an toàn giữa cổng biên (được lưu trữ tại chỗ) và AWS.

## **Tổng quan giải pháp**
Hãy xem xét một dây chuyền sản xuất có các cảm biến thiết bị ghi lại tốc độ dòng chảy, nhiệt độ và áp suất. Để thực hiện phân tích trên dữ liệu này, bạn thu thập dữ liệu truyền phát (streaming data) theo thời gian thực từ các cảm biến này vào môi trường AWS bằng cách sử dụng một cổng biên. Sau khi dữ liệu được đưa vào AWS, bạn có thể sử dụng các dịch vụ phân tích khác nhau để thu thập thông tin chi tiết.

Để minh họa luồng dữ liệu từ biên lên đám mây (cloud), chúng tôi có các tài sản, máy móc và công cụ xuất bản (publish) dữ liệu bằng MQTT. Theo tùy chọn, chúng tôi sử dụng một thiết bị biên mô phỏng để xuất bản dữ liệu đến một MQTT endpoint cục bộ. Chúng tôi sử dụng cổng biên với môi trường runtime AWS IoT Greengrass V2 edge để truyền dữ liệu qua Amazon Data Firehose trên đám mây đến S3 Tables.

Biểu đồ sau đây minh họa kiến trúc của giải pháp.

## ![](/images/3-BlogsTranslated/3.2-Blog2/image_1.jpg) 

Luồng công việc bao gồm các bước sau:

1. Thu thập dữ liệu từ các cảm biến Internet of Things (IoT) và truyền dữ liệu thời gian thực từ các thiết bị biên đến AWS Cloud bằng AWS IoT Greengrass.
2. Thu thập, chuyển đổi và lưu trữ dữ liệu trong thời gian gần thực (near real-time) bằng cách sử dụng Data Firehose, với thành phần Firehose trên AWS IoT Greengrass, và tích hợp S3 Tables.
3. Lưu trữ và sắp xếp dữ liệu dạng bảng bằng cách sử dụng S3 Tables, cung cấp bộ lưu trữ được xây dựng chuyên biệt cho định dạng Apache Iceberg với giải pháp truy vấn đơn giản, hiệu suất cao và tiết kiệm chi phí.
4. Truy vấn và phân tích dữ liệu dạng bảng bằng cách sử dụng Amazon Athena.

Luồng dữ liệu tại biên bao gồm các thành phần chính sau:

*   **Thiết bị IoT đến MQTT broker cục bộ** – Một thiết bị mô phỏng được sử dụng để tạo dữ liệu cho mục đích của bài viết này. Trong một triển khai thực tế điển hình, đây sẽ là thiết bị hoặc cổng của bạn có hỗ trợ MQTT. Các thiết bị IoT có thể xuất bản tin nhắn đến một MQTT broker cục bộ (Moquette) đang chạy trên AWS IoT Greengrass.
*   **Cầu nối MQTT (MQTT bridge)** – Thành phần MQTT bridge chuyển tiếp tin nhắn giữa MQTT broker (nơi các thiết bị máy khách giao tiếp) và publish/subscribe (IPC) cục bộ của AWS IoT Greengrass.
*   **Thành phần PubSub cục bộ (tùy chỉnh)** – Thành phần này đăng ký (subscribe) các tin nhắn IPC cục bộ, chuyển tiếp tin nhắn đến chủ đề (topic) `kinesisfirehose/message`, và sử dụng giao diện IPC để đăng ký nhận tin nhắn.
*   **Thành phần Firehose** – Thành phần Firehose đăng ký vào chủ đề `kinesisfirehose/message`. Sau đó, thành phần này sẽ truyền dữ liệu đến Data Firehose trên đám mây. Nó sử dụng QoS 1 để đảm bảo việc gửi tin nhắn đáng tin cậy.

Bạn có thể mở rộng giải pháp này ra nhiều vị trí biên, giúp bạn có được cái nhìn liền mạch về dữ liệu trên nhiều vị trí của khu vực sản xuất, hoạt động như một giải pháp low-code. 

Trong các phần tiếp theo, chúng tôi sẽ hướng dẫn các bước để cấu hình luồng thu thập dữ liệu đám mây:

1. Tạo một S3 Tables bucket và kích hoạt tích hợp với các dịch vụ phân tích của AWS.
2. Tạo một không gian tên (namespace) trong table bucket bằng AWS Command Line Interface (AWS CLI).
3. Tạo một bảng (table) trong table bucket với schema đã định nghĩa bằng AWS CLI.
4. Tạo một vai trò AWS Identity and Access Management (IAM) cho Data Firehose với các quyền cần thiết.
5. Cấu hình các quyền của AWS Lake Formation bằng cách cấp quyền Super trên các bảng cụ thể cho vai trò Data Firehose.
6. Thiết lập luồng Data Firehose bằng cách chọn Direct PUT làm nguồn và Iceberg tables làm đích. Cấu hình cài đặt đích với tên cơ sở dữ liệu và bảng, chỉ định một Amazon Simple Storage Service (Amazon S3) bucket cho đầu ra lỗi (error output), và liên kết với vai trò IAM đã tạo trước đó.
7. Xác minh và truy vấn dữ liệu bằng Athena bằng cách cấp quyền Lake Formation để Athena truy cập và truy vấn bảng để xác minh việc thu thập dữ liệu.

## **Điều kiện tiên quyết**
Bạn phải đáp ứng các điều kiện tiên quyết sau:
*   Một tài khoản AWS
*   Các đặc quyền IAM cần thiết để khởi chạy AWS IoT Greengrass trên cổng biên (hoặc một thiết bị được hỗ trợ khác)
*   Một phiên bản Amazon Elastic Compute Cloud (Amazon EC2) với hệ điều hành được hỗ trợ để thực hiện proof of concept (bằng chứng khái niệm)

## **Cài đặt AWS IoT Greengrass trên cổng biên**
Để biết hướng dẫn cài đặt AWS IoT Greengrass, hãy tham khảo Cài đặt phần mềm AWS IoT Greengrass Core. Sau khi hoàn tất cài đặt, bạn sẽ có một thiết bị core được cấp phép, như hiển thị trong ảnh chụp màn hình sau. Trạng thái của thiết bị là Healthy (Khỏe mạnh), có nghĩa là tài khoản của bạn có thể giao tiếp thành công với thiết bị.

Đối với proof of concept, bạn có thể sử dụng phiên bản EC2 chạy Ubuntu làm cổng biên của mình.

## ![](/images/3-BlogsTranslated/3.2-Blog2/image_2.jpg) 

## **Cấp phép luồng Data Firehose**
Để biết các bước chi tiết về cách thiết lập Data Firehose gửi dữ liệu đến các bảng Iceberg, hãy tham khảo Gửi dữ liệu đến Apache Iceberg Tables bằng Amazon Data Firehose. Để tích hợp S3 Tables, hãy tham khảo Xây dựng data lake cho dữ liệu truyền phát với Amazon S3 Tables và Amazon Data Firehose.

Bởi vì bạn đang sử dụng AWS IoT Greengrass để truyền dữ liệu, bạn có thể bỏ qua các bước Kinesis Data Generator được đề cập trong các hướng dẫn này. Thay vào đó, dữ liệu sẽ chảy từ các thiết bị biên của bạn qua các thành phần Greengrass đến Data Firehose. 

Sau khi hoàn thành các bước này, bạn sẽ có một luồng Firehose và S3 Tables bucket, như hiển thị trong ảnh chụp màn hình sau. Lưu ý Amazon Resource Name (ARN) của luồng Firehose để sử dụng trong các bước tiếp theo.

## ![](/images/3-BlogsTranslated/3.2-Blog2/image_3.jpg) 

## **Triển khai các thành phần Greengrass**
Hoàn thành các bước sau để cấu hình và triển khai các thành phần Greengrass. Để biết thêm chi tiết, hãy tham khảo Tạo các triển khai.

Sử dụng cấu hình sau để kích hoạt định tuyến tin nhắn từ MQTT cục bộ đến thành phần AWS IoT Greengrass PubSub. Lưu ý chủ đề (topic) trong đoạn mã. Đây là chủ đề MQTT mà các thiết bị sẽ gửi dữ liệu đến.

```json
{
  "reset": [""],
  "merge": {
    "mqttTopicMapping": {
      "HelloWorldIotCoreMapping": {
        "topic": "clients/#",
        "source": "LocalMqtt",
        "target": "Pubsub"
      }
    }
  }
}
```
Sử dụng cấu hình sau để triển khai thành phần Firehose. Sử dụng ARN của luồng Firehose mà bạn đã lưu ý trước đó.

```json
{
  "reset": [""],
  "merge": {
    "lambdaExecutionParameters": {
      "EnvironmentVariables": {
        "DEFAULT_DELIVERY_STREAM_ARN": "arn:aws:firehose:us-east-1:<<account-id>>:deliverystream/<<stream name>>"
      }
    },
    "containerMode": "NoContainer"
  }
}
```
Sử dụng cấu hình sau để triển khai thành phần bộ định tuyến đăng ký cũ (Lưu ý rằng đây là thành phần phụ thuộc của thành phần Firehose):

```json
{
  "reset": [""],
  "merge": {
    "subscriptions": {
      "aws-greengrass-kinesisfirehose": {
        "id": "aws-greengrass-kinesisfirehose",
        "source": "component:aws.greengrass.KinesisFirehose",
        "subject": "kinesisfirehose/message/status",
        "target": "cloud"
      }
    }
  }
}
```
Tạo và triển khai một thành phần PubSub tùy chỉnh. Bạn có thể sử dụng đoạn mã mẫu sau bằng ngôn ngữ ưa thích của mình để triển khai như một thành phần Greengrass. Bạn có thể sử dụng gdk để tạo các thành phần tùy chỉnh.

```json
{
  "reset": [""],
  "merge": {
    "subscriptions": {
      "aws-greengrass-kinesisfirehose": {
        "id": "aws-greengrass-kinesisfirehose",
        "source": "component:aws.greengrass.KinesisFirehose",
        "subject": "kinesisfirehose/message/status",
        "target": "cloud"
      }
    }
  }
}
```
Sau khi triển khai các thành phần, bạn sẽ thấy chúng trên tab Components (Các thành phần) của thiết bị core.

## **Thu thập dữ liệu**
Trong bước này, bạn thu thập dữ liệu từ thiết bị của mình lên AWS IoT Greengrass, sau đó dữ liệu này sẽ được đưa vào Data Firehose. Hoàn thành các bước sau:

1. Từ thiết bị biên có hỗ trợ MQTT hoặc cổng biên của bạn, xuất bản dữ liệu đến chủ đề đã xác định trước đó (client/#). Ví dụ: chúng tôi xuất bản dữ liệu đến chủ đề MQTT client/devices/telemetry.

2. Nếu bạn muốn thực hiện việc này như một proof of concept, hãy tham khảo Tạo thiết bị ảo với Amazon EC2 để tạo một thiết bị IoT mẫu.

Đoạn mã sau là payload mẫu cho ví dụ của chúng tôi:

```json
PAYLOAD="{
\"device_id\": \"$DEVICE_ID\",
\"timestamp\": \"$TIMESTAMP\",
\"temperature\": $TEMPERATURE,
\"pressure\": $PRESSURE,
\"flow_rate\": $FLOW_RATE,
\"vibration\": $VIBRATION,
\"motor_speed\": $MOTOR_SPEED,
\"status\": \"$STATUS\",
\"battery\": $((RANDOM % 30 + 70 )),}"
```

Để biết thêm chi tiết về cách xuất bản tin nhắn từ một thiết bị mẫu, hãy tham khảo Cấp phép tức thời.

Thành phần MQTT bridge sẽ định tuyến payload từ chủ đề MQTT (client/devices/telemetry) đến một chủ đề IPC có cùng tên. Thành phần tùy chỉnh mà bạn đã triển khai trước đó sẽ lắng nghe chủ đề IPC client/devices/telemetry và xuất bản đến chủ đề IPC kinesisfirehose/message. Tin nhắn phải tuân theo cấu trúc được mô tả trong Dữ liệu đầu vào.

## **Xác minh dữ liệu trong Athena**

Giờ đây, bạn có thể truy vấn dữ liệu được xuất bản từ thiết bị IoT biên bằng Athena. Trên bảng điều khiển Athena, tìm danh mục (catalog) và cơ sở dữ liệu mà bạn đã thiết lập và chạy truy vấn sau:

```json
SELECT * FROM <<database>>."device_telemetry" limit 10;
```

Bạn sẽ thấy dữ liệu được hiển thị như trong ảnh chụp màn hình sau. Lưu ý tên cơ sở dữ liệu và bảng mà bạn đã xác định trong bước "Cấp phép luồng Data Firehose".

## **Mở rộng giải pháp**
Trong các phần trước, chúng tôi đã chỉ ra cách nhiều thiết bị có thể đưa dữ liệu lên đám mây bằng cách sử dụng một cổng biên Greengrass duy nhất. Vì các vị trí sản xuất thường phân tán trong kịch bản thế giới thực, bạn có thể thiết lập các thiết bị Greengrass tại các địa điểm khác và xuất bản dữ liệu đến cùng một luồng Firehose. Điều này đảm bảo dữ liệu từ các địa điểm khác nhau sẽ được đưa vào một S3 bucket duy nhất, được phân vùng một cách phù hợp (trong ví dụ của chúng tôi là Device_Id), và có thể được truy vấn một cách liền mạch.

## **Dọn dẹp**
Sau khi xác minh kết quả, bạn có thể xóa các tài nguyên sau để tránh phát sinh thêm chi phí:

Xóa phiên bản EC2 Ubuntu mà bạn đã tạo cho proof of concept.

Xóa luồng phân phối Firehose và các tài nguyên liên quan.

Xóa (Drop) các bảng Athena đã tạo để truy vấn dữ liệu.

Xóa S3 Tables bucket mà bạn đã cấp phép.

## **Kết luận**
Trong bài viết này, chúng tôi đã hướng dẫn cách thiết lập một khuôn khổ thu thập dữ liệu gần thời gian thực từ biên lên đám mây có khả năng mở rộng bằng AWS IoT Greengrass và bắt đầu thực hiện phân tích dữ liệu trong các dịch vụ AWS bằng phương pháp low-code. Chúng tôi đã trình bày cách tối ưu hóa lưu trữ dữ liệu sang định dạng Iceberg với S3 Tables và chuyển đổi dữ liệu truyền phát trước khi nó được đưa vào lớp lưu trữ bằng Data Firehose. Chúng tôi cũng đã thảo luận về cách bạn có thể mở rộng giải pháp này theo chiều ngang qua nhiều địa điểm sản xuất (nhà máy hoặc khu vực) để tạo ra một giải pháp low-code nhằm phân tích dữ liệu theo thời gian gần thực.

Để tìm hiểu thêm, hãy tham khảo các tài nguyên sau:

Hướng dẫn dành cho nhà phát triển Amazon Data Firehose

Làm việc với Amazon S3 Tables và table buckets

Xây dựng data lake cho dữ liệu truyền phát bằng Amazon S3 Tables và Amazon Data Firehose

**Bài đăng trên AWS Study Group**
## ![](/images/image_2.png)

## **Về các tác giả**

<div style="display:flex; flex-direction:column; gap:1rem;">

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.2-Blog2/image_6.jpg" alt="Joyson Neville Lewis" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Joyson Neville Lewis</strong><br/>
      Joyson Neville Lewis là Kiến trúc sư Trí tuệ nhân tạo Đàm thoại (Conversational AI Architect) cấp cao tại AWS Professional Services. Joyson từng làm Kỹ sư Phần mềm/Dữ liệu trước khi lấn sân sang lĩnh vực AI Đàm thoại và IoT Công nghiệp. Anh hỗ trợ các khách hàng của AWS hiện thực hóa tầm nhìn AI của họ bằng cách sử dụng Trợ lý giọng nói/Chatbot và các giải pháp IoT.
    </div>
  </div>

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.2-Blog2/image_7.jpg" alt="Anil Vure" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Anil Vure</strong><br/>
      Anil Vure là Kiến trúc sư Dữ liệu IoT cấp cao tại AWS Professional services. Anil có nhiều kinh nghiệm trong việc xây dựng các nền tảng dữ liệu quy mô lớn và làm việc với các khách hàng trong ngành sản xuất để thiết kế các hệ thống thu thập dữ liệu tốc độ cao.
    </div>
  </div>

  <div style="display:flex; align-items:flex-start; gap:1rem;">
    <img src="/images/3-BlogsTranslated/3.2-Blog2/image_8.jpg" alt="Ashok Padmanabhan" style="width:120px; height:120px; object-fit:cover; border-radius:8px;" />
    <div>
      <strong>Ashok Padmanabhan</strong><br/>
      Ashok Padmanabhan là Kiến trúc sư Dữ liệu IoT cấp cao tại AWS Professional Services. Ashok chủ yếu làm việc với các khách hàng trong ngành sản xuất và ô tô để thiết kế và xây dựng các giải pháp Công nghiệp 4.0.
    </div>
  </div>
  
</div>