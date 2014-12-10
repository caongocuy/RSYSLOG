Cấu hình Syslog server với Rsyslog trên Ubuntu14.04.1
======
Mục Lục

[1. Giới thiệu](#gioithieu)

[2. Cơ bản về syslog](#coban)
				
[3. Mô hình LAB](#mohinhlab)
				
[4. Lab mô hình log với dịch vụ WEB](#web)
	
- [a. Mô hình từ Ubuntu sang Ubuntu](#ubuntu)
	
- [b. Mô hình từ Centos sang Ubuntu](#centos)

- [c. Gửi log từ win sang Ubuntu](#Win)

=====================
<a name="gioithieu"></a>
#### 1. Giới thiệu
- Một máy chủ syslog đại diện cho một điểm giám sát log trên một mạng , mà tất cả các loại thiết bị bao gồm 
cả máy chủ Linux hoặc Windows ... đều có thể gửi các bản ghi log đến đó. Bằng cách thiết lập một máy chủ syslog , 
bạn có thể lọc và củng cố các bản ghi từ máy chủ và các thiết bị khác nhau vào một 
vị trí duy nhất, do đó bạn có thể xem và lưu trữ thông tin quan trọng dễ dàng hơn .

- Trên hầu hết các bản phân phối Linux , rsyslog đã cài đặt sẵn. Cấu hình 
trong một kiến trúc client / server , rsyslog có thể đóng cả hai vai trò ; như một máy chủ syslog rsyslog 
có thể thu thập các bản ghi từ các thiết bị khác , và như là một client syslog , rsyslog có thể truyền 
tải các bản ghi nội bộ của nó đến một máy chủ syslog từ xa .

- Trong hướng dẫn này, tôi giới thiệu cách cấu hình một máy chủ syslog tập trung sử dụng rsyslog trên Linux . Trước khi 
chúng ta đi vào chi tiết, đầu tiên chúng ta cần thống nhất các khái niệm cơ bản của syslog.

<a name="coban"></a>
#### 2. Cơ bản về syslog

Khi các bản ghi được thu thập với cơ chế syslog , ba điều quan trọng phải nắm rõ.
- Facility level: các tiến trình cần ghi log.
- Priority level: loại thông điệp cần thu thập dựa trên mức cảnh báo đưa ra.
- Destination: nơi cần gửi các bản ghi log đến.

###### Facility

Các facility level sẽ xác định các tiến trình nội bộ. Một số Facility trong Linux:
- auth: messages liên quan đến xác thực (login).
- cron: messages liên quan đến lập lịch các processes hoặc applications.
- daemon: messages liên quan đến daemons (internal servers).
- kernel: messages liên quan đến kernel.
- mail: messages liên quan đến mail servers.
- syslog: messages liên quan đến bản thân syslog daemon.
- lpr: messages liên quan đến print servers.
- local0 - local7: messages được định nghĩa bởi người dùng (local7 thường được dùng bởi Cisco hoặc Windows servers).

###### Priority

Mức độ cảnh báo được đưa ra, và xác định bằng cách viết tên của chúng, mức 7 là thấp nhất và cao nhất là mức 0:
- emerg: Emergency - 0 : Khẩn cấp.
- alert: Alerts - 1 : cảnh báo cần phải can thiệp ngay lập tức.
- crit: Critical - 2 : cảnh báo nguy hiểm.
- err: Errors - 3 : tình trạng lỗi, cần chú ý can thiệp.
- warn: Warnings - 4 : tình trạng cảnh báo.
- notice: Notification - 5 : tình trạng yêu cầu cần chú ý.
- info: Information - 6 : thông báo các thông tin đơn giản- không cần can thiệp.
- debug: Debugging - 7 : thông tin gỡ lỗi từ các chương trình.

###### Destination

Cuối cùng cần chỉ ra đích đến của các bản tin trên syslog-client, 
- Lưu log trên local
- Gửi log đến một máy chủ syslog ở xa qua giao thức TCP/UDP
- Gửi stdout đến 1 giao diện điều khiển

Trong Rsyslog, cấu hình syslog dựa trên mô hình sau

    [facility-level].[Priority-level]  [destination]

Ví dụ:

    cron.*              /var/log/cron
	
Dòng này có ý nghĩa tất cả log của deamon cron sẽ được lưu vào /var/log/cron. (*) sau dấu (.) có nghĩa là tất cả các mức Priority 
sẽ được log lại. Tương tự như vậy, nếu thay cron = * thì có nghĩa là tất cả các Facility với tất cả các Priority được log lại trong /var/log/cron.

	mail.warn           /var/log/mail.warn
	
Dòng này có ý nghĩa là với Facility là mail thì tất cả các Priority từ "warning" trở lên (gồm có error, critical....) sẽ được log lại trong /var/log/mail.warn.
	
    mail.=info          /var/log/mail.info
	
Sử dụng dấu (=) đằng sau dấu (.) có ý nghĩa rằng với Facility là mail thì chỉ có Priority = info mới được log lại.
	
	mail.!=info         /var/log/mail.info
	
Chèn thêm dấu (!) như trên, có nghĩa là với Facility là mail thì các Priority có mức độ từ info trở lên sẽ được log lại (ngoại trừ chính info).
	
	*.*					@172.16.69.111:514
	
Dòng này có ý nghĩa, tất cả các Facility và Priority sẽ được chuyển đến máy chủ có ip : 172.16.69.111 theo giao thức UDP và port 514.

###### Các lệnh dùng để xem log trong linux

Đối với các file ghi log các bạn có thể dùng một số lệnh sau để giúp cho việc xem log

|Câu lệnh | Cú pháp |Ý nghĩa | Ghi chú thêm |
|---------|---------|--------|--------------|
|more | more [file] | Dùng xem toàn bộ nội dung của thư muc | Đối với câu lênh này nôi dung được xem theo từng trang. Bạn dung dấu "cách" để chuyển  trang |
|tail | tail [file] | In ra 10 dòng cuối cùng nội dung của file | thêm tùy chọn -n [số dòng] sẽ in ra số dòng theo yêu cầu |
|head | head [file] | In ra 10 dòng đầu tiên của nôi dụng file |
|tail -f | tail -f [file] | Dùng để xem ngay lâp tức khi có log đến | Đây là câu lệnh dùng phổ biến nhất nó giúp ta có thể xem ngay lập tức log mới đến, và nó sẽ in ra 10 dong cuối cùng trong nội dung file đó |

<a name="mohinhlab"></a>
#### 3. Mô hình LAB

Để rõ hơn về cơ chế cũng như cấu hình một mô hình client / server, tôi sẽ đưa ra một vài bài lab cơ bản với dịch vụ Web trên linux, và tôi sẽ giới thiệu 
một tool dùng trên Win để gửi log đến máy chủ Linux.