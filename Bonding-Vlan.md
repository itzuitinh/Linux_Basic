# 1. Bonding là gì
- Bonding là việc kết hợp nhiều NIC thành một NIC logic duy nhất nhằm cân bằng tải, tăng thông lượng, khả năng chịu lỗi, etc. của hệ thống.

# 1.1 Mode
  - Mode 0 - balance-rr: Áp dụng cơ chế Round-robin cung cấp khả năng cân bằng tải và chịu lỗi
  - Mode 1 - active-backup: Áp dụng cơ chế Active-backup. Tại một thời điểm chỉ có 1 slave interface active, các slave khác sẽ active khi nào slave đang active bị lỗi.
  - Mode 2 - balance-xor: Áp dụng phép XOR, thực hiện XOR MAC nguồn và MAC đích, rồi thực hiện modulo với số slave. Mode này cung cấp khả năng cân bằng tải và chịu lỗi
  - Mode 3 - broadcast: Gửi tin trên tất cả các slave interfaces. Mode này cung cấp khả năng chịu lỗi.
  - Mode 4 - 802.3ad: IEEE 802.3ad. Mode này sẽ tạo một nhóm tập hợp các intefaces chia sẻ chung tốc độ và thiết lập duplex (hai chiều). Yêu cầu để sử dụng mode này là có Ethtool trên các drivers gốc để đạt được tốc độ và cấu hình hai chiều trên mỗi slave, đồng thời các switch sẽ phải cấu hình hỗ trợ chuẩn IEEE 802.3ad.
  - Mode 5 - balance-tlb: Cân bằng tải thích ứng với quá trình truyền tin: lưu lượng ra ngoài phân tán dựa trên tải hiện tại trên mỗi slave (tính toán liên quan tới tốc độ). Lưu lượng tới nhận bởi slave active hiện tại, nếu slave này bị lỗi khi nhận gói tin, các slave khác sẽ thay thế, MAC address của đường bond sẽ chuyển sang một trong các slave còn lại.
  - Mode 6 - balance-alb: Cân bằng tài thích ứng: bao gồm cả cân bằng tải truyền (balance-tlb) và cân bằng tải nhận (rlb - receive load balancing) đối với lưu lượng IPv4. Cân bằng tải nhận đạt được nhờ kết hợp với ARP. Bondin driver sẽ chặn các bản tin phản hồi ARP gửi bởi hệ thống cụ bộ trên đường ra và ghi đè địa chỉ MAC nguồn bằng địa chỉ MAC của một trong các slaves trên đường bond.

# 2. Cài đặt bonding
- Kiểm tra bonding trong kernel module `lsmod | grep bonding`
- Nếu chưa có thì gõ lệnh sau: 

  `echo 'bonding' >> /etc/modules`

  `modprobe bonding`

- Sau đó cài gói ifenslave ` apt-get install ifenslave`
- Kiểm tra card mạng nào có thể dùng `ifconfig -a`, ở đây ta sẽ dùng `ens38` và `ens39`

- Mở và sửa file `sudo vi /etc/netplan/50-cloud-init.yaml`

  <img src="https://i.imgur.com/eVn8dIx.png">
  
  - bond0: tên card
  - mode: balance-rr: là chọn chế độ mode 0
  - mii-monitor-interval: 100: Xác định mức độ thường xuyên, tính bằng milli giây, bond được kiểm tra nếu nó vẫn đang hoạt động.
  
- Lưu lại cấu hình file `sudo netplan apply`
- Kiểm tra thông tin bond0: `cat /proc/net/bonding/bond0` hoặc dùng lệnh `ifconfig -a` và `ip a`

  <img src="https://i.imgur.com/nggBFrA.png">
  
  
# 3.VLAN
- Vlan là viết tắt của Virtual Local Area Network, hay còn gọi là mạng LAN ảo.
- Cài đặt `apt-get install vlan`
- Kiểm tra kernel module liên quan đến VLAN ` lsmod | grep 8021`
- Nếu chưa được bật, thì bật bằng lệnh: `modprobe 8021q`
- Ta sẽ tạo 2 VLAN có ID là 10 và 20 trên card bond0 vừa tạo ở trên
- Mở và sửa file `sudo vi /etc/netplan/50-cloud-init.yaml` 

    <img src="https://i.imgur.com/5S83kSZ.png">
  
- Lưu lại cấu hình netplan: `netplan apply`
- Kiểm tra `ip a` hoặc `ifconfig -a`
  
    <img src="https://i.imgur.com/ZkT1bbg.png">
  
  
