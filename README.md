Configure Syslog server use Rsyslog in Ubuntu14.04.1
======
Mục Lục

[1. Giới thiệu](#gioithieu)

[2. Cơ bản về syslog](#coban)
				
[3. Mô hình LAB](#mohinhlab)
				
[4. Lab mô hình log với dịch vụ WEB](#web)
	
- [a. Mô hình từ Ubuntu sang Ubuntu](#ubuntu)
	
- [b. Mô hình từ Centos sang Ubuntu](#centos)

- [c. Gửi log từ win sang Ubuntu](#win)

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

###### File cấu hình Rsyslog

*Đối với Centos*

Các rsyslog daemon được định nghĩa trong file *rsyslog.conf*, nằm trong thư mục */etc/*.

Về cơ bản, file *rsyslog.conf* cho phép định nghĩa nơi các rsyslog daemon (mail, cron, auth, local0 ....) lưu log.

Các hoạt động cấu hình rsyslog trên *Centos* đều thực hiện trong file này *rsyslog.conf*, bao gồm cả việc cấu hình rsyslog như một Log server remote.

```
# rsyslog v5 configuration file
...  
...    
# Include all config files in /etc/rsyslog.d/  
IncludeConfig /etc/rsyslog.d/*.conf  

#### RULES ####  
# Log all kernel messages to the console.  
# Logging much else clutters up the screen.  
#kern.*  /dev/console  

# Log anything (except mail) of level info or higher.  
# Don't log private authentication messages!  
*.info;mail.none;authpriv.none;cron.none                /var/log/messages  

# The authpriv file has restricted access.  
authpriv.*                                              /var/log/secure  

# Log all the mail messages in one place.  
mail.*                                                  -/var/log/maillog  


# Log cron stuff  
cron.*                                                  /var/log/cron  

# Everybody gets emergency messages  
*.emerg                                                 *  

# Save news errors of level crit and higher in a special file.  
uucp,news.crit                                          /var/log/spooler  

# Save boot messages also to boot.log  
local7.*                                                /var/log/boot.log  
...  
... 

```

*Đối với Ubuntu*

File dùng cho việc định nghĩa nơi các *rsyslog daemon* lưu log thường là */etc/rsyslog.d/50-default.conf*, ngoài file này bạn cũng có thể định nghĩa ra các file khác tương tự trong thư mục *rsyslog.d/*.

Việc cấu hình rsyslog server được thực hiện trong file *rsyslog.conf*

*Khi các bản ghi được thu thập với cơ chế syslog , ba điều quan trọng phải nắm rõ*

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

*Chú ý 1: @ IP máy chủ log* khi đó các log của mail cũng sẽ gửi đến ip của máy chủ log với port 514 và dùng giao thức UDP. Các bạn nhớ máy chủ log mở port 514 với kiểu truyền vận UPD hay TCP thì trên client cũng phải truyền đúng với giao thức như trên server.

*@IPserver:514* : Đối với giao thức UDP

*@@IPserver:514* : Đối với giao thức TCP

Trong file cấu hình các bạn có thể nhìn thấy một vài dòng có định dạng như *-/var/log/kern.log*, vậy dấu trừ (-) ở phía trước đường dẫn thư mục
có ý nghĩa là gì.

    kern.*                          -/var/log/kern.log

các bạn có thể tìm thấy nó ở [đây](http://www.rsyslog.com/doc/master/compatibility/v3compatibility.html). Đại ý của nó là : rsyslog muốn giữ sự tương thích với syslog nên mặc định nó sẽ đồng bộ các file nếu không có quy định nào khác *(bằng việc để 
dấu trừ (-) phía trước đường dẫn file log đầu ra)*.

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

Để rõ hơn về cơ chế cũng như cấu hình một mô hình client / server. Theo mô hình dưới một máy chủ rsyslog có thể nhận log của nhiều loại client 
nhưng tôi sẽ trình bày theo cách tách nhỏ ra(1-1) thành nhiều kịch bản, tôi sẽ đưa ra một vài bài lab cơ bản với dịch vụ Web trên linux, và tôi sẽ giới thiệu 
một tool dùng trên Win để gửi log đến máy chủ Linux.

![img](http://i.imgur.com/CZB7Zyh.png "img")

<a name="ubuntu"></a>

###### a. Mô hình từ Ubuntu sang Ubuntu

Lúc này máy Ubuntu chạy dịch vụ http và là máy client gửi bản log về cho máy Ubuntun là log server

- Bước 1: Chỉnh sửa trong file cấu hình `/etc/rsyslog.conf` của máy chủ Log-server để nó có thể nhận các bản tin log từ các client gửi về.

<img class="image__pic js-image-pic" src="http://i.imgur.com/667Q082.png" alt="" id="screenshot-image">

Nếu bạn muốn trên máy chủ log tạo thành các thư mục lưu riêng log đối với từng máy Client gửi về thêm dòng này vào file cấu hình *rsyslog.conf*. Việc định nghĩa ra các template như thế này, nhằm mục đích quản lí log của 
các máy client tốt hơn (vì mỗi client là một thư mục), nếu không log của client sẽ được báo hết lại trên cùng 1 file tương ứng ở server, lượng client lớn thì điều này làm hạn chế việc phân tích log rất nhiều.

Bạn add thêm các dòng sau vào cuối file *rsyslog.conf*

```
$template TmplAuth,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
*.*     ?TmplAuth
```

Và chuyển chủ sở hưu tập tin /log/var cho syslog để nó có thể tạo các file và thư mục trong /var/log
```
chown syslog.syslog /var/log
```


- Bước 2: Thêm  dòng này trong file cấu hình `/etc/rsyslog.conf` của máy Client 
```
*.*			@ [Địa chỉ IP của máy log-server]:514
```

- Bước 3: 

Thêm dòng sau đây vào file cấu hình của apache trong máy client: `/etc/apache2/apache2.conf`
```
ErrorLog syslog:local1
```
Thêm dòng sau vào file `/etc/apache2/sites-enabled/000-default.conf`
```
CustomLog "| /usr/bin/logger -thttpacces -plocal1.info"
```
*Note: Dòng lệnh trên có ý nghĩa chuyển tất cả các Log của phần CustomLog vào đầu vào lệnh logger và lệnh logger cho ra đầu ra của log với nguồn là httpacces và với selector là local1.info. Bạn có thể tìm hiểu thêm lệnh logger tại [đây](http://linux.about.com/library/cmd/blcmdl1_logger.htm)*

Lúc này trên máy chủ log nó sẽ tạo ra một thư mục có tên của máy client và trong đó sẽ chứa tất cả các log của máy Client đó.

<img class="image__pic js-image-pic" src="http://i.imgur.com/EXhzms9.png" alt="" id="screenshot-image">

<a name="centos"></a>

###### b. Mô hình Centos sang Ubuntu

Lúc này máy Centos chạy dịch vụ http và là máy client gửi bản log về cho máy Ubuntun là log server

Phần cấu hình trên máy log server hoàn toàn như cấu hình đối với mô hình Ubuntu sang Ubuntu

Phần cấu hình trên máy Client là Centos

- Bước 1: Chỉnh sửa file `/etc/ryslog.conf`:thêm dòng sau đây vào file cấu hình syslog
```
*.* 		@ <địa chỉ ip của log server>:514
```

- Bước 2: Chỉnh sửa file cấu hình của http `/etc/httpd/conf/httpd.conf`

Ban thêm 2 dòng sau đây vào file cấu hình
```
ErrorLog syslog:local2
CustomLog "| /usr/bin/logger -thttp_acces -plocal2.info" combined
```
*Chú ý 1:* Bạn hay chắc chắn rằng trong file cấu hình của httpd chỉ có duy nhất dòng ErrorLog bạn vừa nhâp, những ErrorLog khác bạn hãy chuyển thành comment.

*Chú ý 2:* Chúng tôi đang lab với trường hợp tắt iptables. CÒn nếu bạn muốn lab trong trường hợp bạn bật iptables lên, hãy chắc chắn bạn hiểu về iptables nhé.

<a name="win"></a>

###### c. Gửi log từ win sang ubuntu

Để chuyển tiếp các bản tin log từ một máy Windows tới một máy chủ rsyslog, chúng ta cần một Windows syslog agent. Có rất nhiều các 
agent có thể chạy trên Windows, ở đây tôi dùng [Datagram SyslogAgent](http://www.syslogserver.com/download.html) , đó là một phần mềm miễn phí.

Sau khi tải về và cài đặt các agent, chúng ta cần phải cấu hình nó để chạy như một dịch vụ. Xác định giao thức, ip, cổng của máy chủ rsyslog từ xa, và những loại bản ghi được truyền đi.

![img](http://i.imgur.com/YG8dw0g.png "img") 

Sau khi thiết lập xong, chúng ta đã có thể bắt đầu dịch vụ và xem các tập tin log trên máy chủ rsyslog.

Tài liệu tham khảo

[Link trang chủ của syslog](http://www.rsyslog.com/doc/master/index.html)

[Link 1](https://www.digitalocean.com/community/tutorials/how-to-view-and-configure-linux-logs-on-ubuntu-and-centos)

[Link 2](http://xmodulo.com/configure-syslog-server-linux.html)