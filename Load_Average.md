# 1. Load Average là gì:
- Load Average – tạm dịch là “giá trị tải trung bình” – là một chỉ số liên quan đến CPU rất cơ bản và quan trọng. Việc nắm rõ ý nghĩa của chỉ số này giúp chúng ta đánh giá được hiệu năng hiện thời của máy tính cũng như sử dụng CPU nói riêng, máy tính nói chung một cách hiệu quả nhất

- Để kiểm tra load average, chúng ta dùng lệnh `uptime`

  - Đây chính là 3 chỉ số load average của CPU trong 1 phút - 5 phút - 15 phút
  
# 2. Ví dụ về Load Average 
- Giả sử chúng ta có một cái cầu bắt qua sông, có một làn xe duy nhất và có thể chứa 10 chiếc xe. Sẽ có các trường hợp sau:
    - Loadavg của chiếc cầu đang là 0.00: có nghĩa là cầu đang trống.
    - Loadavg của chiếc cầu đang là 1.00: có nghĩa là cầu đang hoạt động 100% sức chứa của nó.
    - Loadavg hơn 1.00: có nghĩa là có đủ 1 làn xe đang đi trên cầu và 1 vài xe đang đợi đi sang cầu.
- Load average cũng giống như các tiến trình(process) đang sử dụng CPU và những tiến trình đã vào queue chờ để dùng CPU. Như cái cầu ở ví dụ trên, thì loadavg phải dưới 1.00, có thể trên 1.00 vài thời điểm, nếu trên 1.00 quá lâu thì bạn nên bắt đầu lo lắng.

- Trên hệ thống nhiều CPU Cores, thì load average có liên hệ với số lượng CPU Cores đang có trên hệ thống. "Dùng hết 100%" là loadavg 1.00 trên hệ thống 1 CPU core, 2.00 trên hệ thống Dual Core, 4.00 trên hệ thống Quad-core...vv. Cũng giống như ví dụ về cái cầu bên trên, nếu chúng ta xây cái cầu có 2 làn xe, thì lúc đó loadavg của cầu là 1.00 thì có nghĩa cầu mới dùng được 50% sức chứa của nó.

- Một vài phân tích về Load Average:
  - Nếu loadavg 0.00: hệ thống rãnh rỗi (idle) hoàn toàn.
  - Nếu loadavg của một phút cao hơn năm phút hoặc mười lăm phút thì loadavg đang tăng, ngược lại thì giảm.
  - Nếu loadavg cao hơn số CPU Cores mà hệ thống đang có thì hệ thống đang gặp vấn đề.
- Nhưng thực tế Load Average trên Linux hiện tại không có đơn giản và rõ ràng vì nó bao gồm khá nhiều nhóm tài nguyên khác nhau (lượng RAM sử dụng, lượng ổ cứng còn, %CPU đang load,...)

# 3. Các yếu tố ảnh hưởng tới Load Average
- Tải của hệ thống có thể được tính toán dựa trên các tiến trình đang được xử lý và các tiến trình đợi xử lý, ngoài ra còn bao gồm các tiến trình uninterruptible sleep states (waiting disk I/O hoặc network). Những tiến trình này cũng góp phần làm tăng cao tải hệ thống mặc dù nó không thực sự sử dụng CPU
- Có 3 yếu tố góp phần làm tăng tải hệ thống:

  - CPU Utilazion: Trong trường hợp CPU được dùng cho những tính toán rất nhẹ nhàng có thể xong tức thì nhưng số lượng process cần CPU lại rất cao, các process cần xử lý tại một thời điểm vượt mức CPU core server hiện có.
  
  - Disk I/O: là giá trị thời gian mà một CPU (hoặc tất cả các CPU) idle bởi vì các tiến trình Runnable đang chờ đợi một hoạt động I/O disk được hoàn thành trong khoảng thời gian nhất định. VD: Máy chủ DB dành thời gian chủ yếu đợi thao tác vào ra (I/O) như khi truy vấn cơ sở dữ liệu. Số lượng query lớn, số lượng truy vấn cần sắp xếp lớn nhưng dữ liệu cần sắp xếp lại rất bé, thời gian đợi dữ liệu từ disk lại cao. Vì vậy phần lớn CPU sẽ idle, nhưng loadavg vẫn cao.
  
  - Network Traffic
  
- Để tăng mức Load Average ta có thể sử dụng nhiều chương trình để tăng lượng sử dụng CPU, thực hiện traffic lớn...
  



