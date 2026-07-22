---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Chrono Genesis Game  
## Web game đối kháng (turn – base trading cards game) được xây dựng theo kiến trúc Serverless trên nền tảng AWS

### 1. Tóm tắt điều hành  
Dự án là Chrono Genesis Game mang thể loại Web game đối kháng (turn – base trading cards game) được xây dựng theo kiến trúc Serverless Real-time Architecture trên nền tảng AWS.

Hệ thống sử dụng đồng thời Amazon API Gateway (HTTP API) và Amazon API Gateway (WebSocket API). HTTP API đảm nhiệm các chức năng truyền thống như đăng nhập, quản lý hồ sơ người chơi, bộ bài và lịch sử trận đấu; trong khi WebSocket API chịu trách nhiệm đồng bộ trạng thái trận đấu theo thời gian thực. Các nghiệp vụ của game được xử lý bởi nhiều AWS Lambda chuyên biệt. Amazon DynamoDB đóng vai trò là trung tâm dữ liệu, lưu trữ trạng thái trận đấu (Game State), thông tin kết nối WebSocket, dữ liệu người dùng, bộ bài và lịch sử trận đấu với độ trễ thấp và khả năng mở rộng cao.


### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  
- Các hệ thống game thời gian thực truyền thống yêu cầu chi phí duy trì máy chủ liên tục dù không có người chơi.

- Độ trễ mạng ảnh hưởng tiêu cực đến trải nghiệm của tựa game mang tính chiến thuật tính toán liên tục.

*Giải pháp*  
Triển khai kiến trúc Serverless Real-time Architecture sử dụng Amazon API Gateway (HTTP API) và Amazon API Gateway (WebSocket API). Toàn bộ game logic được xử lý bởi các AWS Lambda độc lập. Sau khi trận đấu kết thúc, các tác vụ không yêu cầu phản hồi tức thời sẽ được đưa vào Amazon SQS để xử lý bất đồng bộ, giúp giảm tải Game Engine và tối ưu hiệu năng hệ thống.

### 3. Kiến trúc giải pháp  
Nền tảng áp dụng kiến trúc AWS Serverless để vận hành ứng dụng Web Game đối kháng thẻ bài thời gian thực, có khả năng tự động mở rộng quy mô đáp ứng hàng nghìn người chơi đồng thời. Giao diện người dùng được triển khai trên AWS Amplify và truy cập thông qua Amazon Route 53. Người chơi đăng nhập bằng Amazon Cognito để nhận JWT Token. Các yêu cầu REST (đăng nhập, hồ sơ, bộ bài, lịch sử...) được gửi đến Amazon API Gateway (HTTP API), trong khi các thao tác trong trận đấu sử dụng Amazon API Gateway (WebSocket API) nhằm đảm bảo truyền dữ liệu hai chiều với độ trễ thấp.

Các Lambda chuyên biệt như Cancel Match, Start Match, Process Game Engine, Save Deck, Handle Timeout và End Match xử lý toàn bộ nghiệp vụ của trò chơi. Process Game Engine đóng vai trò trung tâm, cập nhật trạng thái trận đấu vào Amazon DynamoDB và phát kết quả đến người chơi thông qua WebSocket.

Sau khi trận đấu kết thúc, Lambda End Match gửi sự kiện vào Amazon SQS. Lambda Post Match Worker đọc dữ liệu từ hàng đợi để xử lý các tác vụ hậu kỳ như cập nhật Rank, EXP, Match History và Game Logs. Cơ chế bất đồng bộ này giúp Game Engine phản hồi nhanh hơn, giảm thời gian chờ của người chơi và tăng khả năng mở rộng hệ thống.

![Architecture](/images/2-Proposal/arch.png)


*Dịch vụ AWS sử dụng*  
- *AWS Amplify*: Lưu trữ và phân phối giao diện web game (React/TypeScript), tự động hóa CI/CD.

- *AWS Lambda*: Xử lý dữ liệu và tách nghiệp vụ.

- *Amazon Route 53*: Giao tiếp với ứng dụng web.  

- *Amazon Cognito*: Xác thực danh tính người chơi, quản lý phiên đăng nhập và cấp phát JWT Token. 

- *Amazon API Gateway (HTTP API)*: Cung cấp các REST API phục vụ đăng nhập, quản lý hồ sơ người chơi, bộ bài, lịch sử trận đấu và các chức năng ngoài thời gian thực.

- *Amazon API Gateway (WebSocket API)*: Duy trì kết nối hai chiều thời gian thực giữa người chơi và Game Engine, truyền nhận dữ liệu của trận đấu với độ trễ thấp.

- *AWS Lambda*: Xử lý game logic tập trung và các tác vụ tính toán.

- *Amazon SQS*: Hàng đợi bất đồng bộ nhận dữ liệu từ Lambda Engine và xử lý.

- *Amazon DynamoDB*: Cơ sở dữ liệu NoSQL lưu trữ trạng thái trận đấu, kết nối và dữ liệu người dùng.

- *Security & Monitoring (IAM, KMS, Secrets Manager, CloudWatch, X-Ray)*: Bảo mật phân quyền, quản lý khóa theo dõi nhật ký và giám sát hiệu năng hệ thống.

*Thiết kế thành phần*  
- *Định tuyến real-time*: Amazon API Gateway kết hợp Route 53 quản lý kết nối WebSocket hai chiều giữa người chơi và hệ thống.   

- *Xử lý game logic*: Các Lambda chuyên biệt (Cancel Match, Start Match, Process Game Engine, Save Deck, Handle Timeout và End Match) phối hợp xử lý toàn bộ vòng đời của một trận đấu.

- *Xử lý bất đồng bộ*: Amazon SQS nhận sự kiện kết thúc trận đấu để Lambda worker tự động tính toán Rank, EXP và lưu lịch sử.

- *Xử lý dữ liệu*: Amazon DynamoDB lưu trạng thái bàn cờ, thông tin kết nối và hồ sơ người chơi.  

- *Giao diện web*: Xây dựng bằng React / TypeScript, đóng gói và phân phối qua mạng lưới CDN của AWS Amplify.

- *Quản lý người dùng*: Sử dụng Amazon Cognito User Pool để quản lý toàn bộ chu trình của tài khoản (đăng ký, xác thực, đổi mật khẩu và thu hồi phiên đăng nhập).

### 4. Triển khai kỹ thuật  
*Các giai đoạn triển khai*  
1. Khởi tạo hạ tầng: Triển khai môi trường, tên miền và thiết lập CI/CD thông qua AWS Amplify.

2. Kết nối & Xác thực: Cấu hình Amazon Cognito, triển khai API Gateway (HTTP API) cho các REST API và API Gateway (WebSocket API) cho giao tiếp thời gian thực.

3. Xây dựng Game Engine: Lập trình các hàm Lambda lõi (Start Match, Process Action, End Match)
xử lý logic thẻ bài.

4. Hậu kỳ trận đấu: Cấu hình Amazon SQS và Lambda Post Match Worker để xử lý cập nhật Rank, EXP, Match History và Game Logs theo cơ chế bất đồng bộ, đảm bảo Game Engine luôn phản hồi nhanh.

5. Kiểm thử & Tối ưu: Giám sát X-Ray, CloudWatch, tối ưu bảo mật với WAF/IAM và thực hiện kiểm thử tải (Stress Test).

*Yêu cầu kỹ thuật*  
- *Hạ tầng hệ thống*: AWS Amplify (Hosting & CI/CD), GitHub, Route 53 (tên miền), IAM và VPC để triển khai, quản lý và bảo mật hệ thống.  

- *Nền tảng game*: Amazon Cognito (xác thực JWT), Amazon API Gateway (HTTP API) phục vụ REST API, Amazon API Gateway (WebSocket API) phục vụ kết nối thời gian thực, AWS Lambda (Cancel Match, Start Match, Process Game Engine, Save Deck, Handle Timeout, End Match, Post Match Worker), Amazon SQS xử lý bất đồng bộ, DynamoDB lưu GameState, UserProfile, Connections, MatchHistory và GameLogs, CloudWatch và X-Ray (giám sát), AWS WAF (bảo mật). Frontend sử dụng React kết nối WebSocket để đồng bộ trạng thái trận đấu theo thời gian thực.

### 5. Lộ trình & Mốc triển khai  
- *Trước thực tập (Tháng 0)*: 1 tháng lên kế hoạch.
    - Tháng 1: Tìm hiểu và học các dịch vụ AWS, thực hành các bài Lab để cũng cố kiến thức.  
    - Tháng 2: Thiết kế và điều chỉnh kiến trúc.  
    - Tháng 3: Triển khai, kiểm thử, đưa vào sử dụng.  
- *Sau triển khai*: Nghiên cứu và phát triển thêm các chức năng mới. 

### 6. Ước tính ngân sách   

*Chi phí hạ tầng*  
- AWS Amplify: 0,00 - 0,02 USD/tháng (Hosting khoảng 500 MB, CI/CD vài lần triển khai, nằm trong Free Tier 12 tháng).

- Amazon Route 53: 0,50 USD/tháng (01 Hosted Zone, chưa tính phí tên miền).

- Amazon Cognito: 0,00 USD/tháng (≤ 10 người dùng, nằm trong Free Tier MAU).

- Amazon API Gateway (WebSocket): 0,00 - 0,02 USD/tháng (~20.000 WebSocket messages và thời gian kết nối thấp, trong Free Tier).

- AWS Lambda: 0,00 USD/tháng (~50.000 requests, 512 MB, dưới Free Tier 1 triệu requests và 400.000 GB-s).

- Amazon DynamoDB: 0,00 USD/tháng (≈1 GB dữ liệu, chọn chế độ Provisioned 25 WCU/RCU để đạt 0 USD trong Free Tier).

- Amazon SQS: 0,00 USD/tháng (~10.000 messages, dưới Free Tier).

- Amazon CloudWatch: 0,00 USD/tháng (≈1 GB log lưu trữ, dưới ngưỡng 5 GB miễn phí/tháng).

- AWS X-Ray: 0,00 USD/tháng (khối lượng trace thấp, trong Free Tier).

- AWS KMS: 0,00 USD/tháng (sử dụng khóa do AWS quản lý).

- AWS Systems Manager Parameter Store: 0,00 USD/tháng (Thay thế Secrets Manager để lưu 01 Secret hoàn toàn miễn phí).

- Truyền dữ liệu Internet: 0,00 USD/tháng (~1 GB Data Transfer Out, dưới ngưỡng 100 GB miễn phí/tháng).

- AWS WAF: 0,00 USD/tháng (nếu tạm tắt) / ≥ 5,00 USD/tháng (nếu bật 01 Web ACL để lọc các request độc hại)

*Tổng chi phí*:
- Chi phí hạ tầng MVP (Chưa tính WAF): khoảng 0,54 USD/tháng (~6,5 USD/năm).

- Chi phí hạ tầng MVP (Có WAF): khoảng 5,54 USD/tháng (~66,5 USD/năm).    

### 7. Đánh giá rủi ro  
*Ma trận rủi ro*  
- Lambda Cold Start gây giật lag lượt đi đầu: Khả năng xảy ra trung bình, mức độ ảnh hưởng trung bình.  

- Mất kết nối mạng WebSocket từ phía người chơi: Khả năng xảy ra cao, mức độ ảnh hưởng cao.  

- Chạm giới hạn AWS Quota: Khả năng xảy ra thấp, mức độ ảnh hưởng rất cao.

*Chiến lược giảm thiểu*  
- Giảm thiểu Lambda Cold Start: Tối ưu thời gian khởi động bằng cách giảm kích thước package, tái sử dụng kết nối và chỉ cấu hình Provisioned Concurrency cho các Lambda xử lý thời gian thực (Process Game Engine) khi hệ thống có lưu lượng truy cập cao. Giải pháp này giúp giảm đáng kể độ trễ ở lượt đi đầu tiên nhưng vẫn tối ưu chi phí vận hành.

- Giảm thiểu mất kết nối WebSocket: Xây dựng cơ chế Auto-Reconnect ở Frontend kết hợp gửi cảnh báo định kỳ để phát hiện mất kết nối. Khi người chơi kết nối lại, API Gateway và Lambda cập nhật Connection ID mới vào DynamoDB, sau đó đồng bộ lại Game State hiện tại để người chơi tiếp tục trận đấu mà không cần tạo phiên mới.

- Giảm thiểu chạm giới hạn AWS Quota: Thiết lập CloudWatch Metrics và CloudWatch Alarms để giám sát số lượng kết nối WebSocket, Lambda Invocations và các tài nguyên quan trọng. Khi tài nguyên đạt khoảng 70–80% giới hạn, hệ thống gửi cảnh báo qua email để quản trị viên chủ động yêu cầu tăng hạn mức trước khi ảnh hưởng đến người dùng. 

*Kế hoạch dự phòng*  
- Mở rộng tài nguyên: Khi tài nguyên AWS tiệm cận Service Quota, quản trị viên yêu cầu tăng hạn mức và tạm thời giới hạn tạo trận đấu mới, ưu tiên tài nguyên cho các trận đang diễn ra nhằm đảm bảo tính ổn định của hệ thống.  

### 8. Kết quả kỳ vọng  
- *Cải tiến kỹ thuật*: Xây dựng thành công luồng Game Engine hoàn toàn trên nền tảng Serverless, thay thế máy chủ duy trì liên tục giúp tiết kiệm chi phí. Sử dụng Amazon SQS để tách biệt các tác vụ hậu kỳ khỏi luồng xử lý thời gian thực, giúp giảm độ trễ, tăng khả năng mở rộng và tối ưu chi phí vận hành.

- *Giá trị dài hạn*: Nền tảng dữ liệu dùng để phát triển game, có thể tái sử dụng cho các dự án tương lai.