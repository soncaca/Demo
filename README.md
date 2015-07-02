Hôm nay, em xin trình báy cách mà một DHCP server hoạt động.
Các bước câu hình một dhcp server mặc định là đã được cài và để bắt các gói tin hoạt động của DHCP mình dùng tcpdump với câu lệnh
` sudo tcpdump -vnes0 -i eth0 port 67 or port 68`
các thông số gồm có
```sh
-v: cho biết nhiều thông tin hơn về gói tin
-n: vô hiệu hóa phân giải tên như vậy bạn không phải chờ đợi trên những phản hồi DNS để hiển thị các gói tin.
-e: hiển thị thông tin lớp liên kết (MAC Address)
-s:  thiết lập số lượng gói tin xuất hiện. 0 có nghĩa là hiển thị đầy đủ gói tin
-i:  chọn giao diện card sử dụng.
DHCP hoạt động trên cổng 67( Server ) và cổng 68 ( Client). Vì vậy, chúng ta có thể nắm bắt các lưu lượng dịch chuyển của nó.
```
Gồm có 4 bước :
####  Bước 1:  The DISCOVER packet
<img src="http://i.imgur.com/WSK66g5.jpg">
Client khởi động  cho phép liên lạc với máy chủ DHCP bằng giao thức TCP/IP. 
Các gói tin bắt đầu với một dấu thời gian. Kể từ khi chúng đang hiển thị thông tin lớp liên kết, các bit tiếp theo là MAC người gửi và MAC địa chỉ đích .  Bạn có thể thấy rằng địa chỉ MAC đích là tất cả đề là f. Điều này có nghĩa nó là một gói tin broadcast. Bởi vì người gửi không biết cụ thể ai để hỏi thông tin DHCP của nó, nó hét lên để mọi người có thể nghe thấy.
**Giống như là khi bạn ra sân bay đón em gái người yêu chẳng hạn, bạn hét lên ai là em gái của X anh là S đến đón em đây,chị gái em bận rồi :)) để họ có thể nhận ra bạn**

Các bit thông tin tiếp theo về các giao thức được sử dụng trong gói tin này. Chúng ta có thể thấy rằng nó là một gói tin IPv4 và UDP của nó (giao thức 17). Các phần tiếp theo chứa các địa chỉ người gửi IP. Tại thời điểm này client không có một địa chỉ IP chính thức vì vậy tất cả là 0 (0.0.0.0 là một địa chỉ IP đặc biệt, dùng để chỉ tất các địa chỉ IP mà một máy đang có). Và kể từ khi người gửi đang quảng bá các gói tin, các ip đích là 255.255.255.255 là địa chỉ IP quảng bá thuộc lớp 3( vì tất cả các thiết bị trên mạng giao tiếp với thiết bị khác ở lớp 3 sử dụng địa chỉ IP, bởi vì địa chỉ MAC hoạt động ở lớp 2 của mô hình OSI, nên chúng chỉ có thể hiểu được các địa chỉ MAC, các địa chỉ lớp 2, hay còn gọi là địa chỉ MAC, được sử dụng để kết hợp với địa chỉ lớp 3 của hệ thống mà bạn đang sử dụng. Địa chỉ lớp 3 còn được gọi với cái tên khác là địa chỉ IP. Khi thiết bị không biết địa chỉ MAC, thì chúng sẽ sử dụng địa chỉ IP để có thể chuyển tiếp lưu lượng). Chúng ta cũng có thể thấy trong phần này người gửi đang sử dụng cổng 68 cố gắng để đến được một máy chủ trên cổng 67, điều này được như mong đợi.

Trong hình có các gói thông tin tiêu đề, chúng ta có tất cả các tùy chọn  đang sử dụng. Chúng ta biết nó là một DISCOVER từ Tùy chọn 53. Chúng ta có thể xem tất cả các thông tin DHCP chuẩn có thể sẽ được yêu cầu (Option 55), tên máy (lựa chọn 12) và rất nhiều thông tin hữu ích khác để chẩn đoán một vấn đề.
####  Bước 2: The OFFER packet
<img src="http://i.imgur.com/g4Q3bgx.jpg">
Mọi máy chủ DHCP có thể nhận thông điệp và chuẩn bị địa chỉ IP cho client, nếu có một DHCP server nhận được gói tin DHCPDISCOVER của client thì nó sẽ trả lời lại bằng một gói tin DHCPOFFER.

Bắt đầu với một dấu thời gian sau đó là địa chỉ MAC của máy chủ DHCP tiếp theo là địa chỉ MAC của client. Vì máy chủ biết ai để gửi gói tin này đến, nó sẽ unicast và thiết lập đích tới client. Vẫn còn một gói tin IPv4 UDP. Các gói tin bắt nguồn từ máy chủ và nó sử dụng địa chỉ IP mà nó cung cấp như là điểm đến của nó.

Option 53 chỉ ra rằng đây là gói OFFER. Các tùy chọn được cung cấp trong gói có chứa một số thông tin rất hữu ích như thời gian thuê trong vài giây (Option 51) DNS Server (Option 6), Subnet Mask (Option 1), Default Gateway (Option 3) và các địa chỉ IP của client có thể sử dụng, địa chị IP phù hợp.
####  Bước 3:  The REQUEST packet.
<img src="http://i.imgur.com/IGiYU01.jpg">
DHCP client đã nhận được gói tin DHCPOFFER thì nó sẽ phản hồi broadcast lại một gói DHCPREQUEST để chấp nhận cái offer đó. Client vẫn không có một địa chỉ IP mặc dù đã xác nhận. Và tất cả các Option để xác nhận hiển thị lên ví dụ như xác nhận các máy chủ DHCP (Option 54), xác minh các địa chỉ IP được sử dụng (Option 50) và như vậy, option 53 cho thấy đây là các gói tin REQUEST.

####  Bước 4: The ACK packet
<img src="http://i.imgur.com/tkmjueR.jpg">
DHCP server nhận được DHCPREQUEST sẽ gởi trả lại DHCP client một DHCPACK cho biết là đã chấp nhận DHCP client đó thuê IP address đó.
Gói tin cuối này có tất cả mọi thứ đầy ra như bạn mong đợi trong header. Và trong các option  có có chỉ ra đây là các gói tin ACK (Option 53).Gói tin này bao gồm địa chỉ IP và các thông tin cấu hình khác ...
#### Kết quả:
<img src="http://i.imgur.com/hTz81s8.jpg">
