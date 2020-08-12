# 1. LVM là gì
- Logical Volume Management(LVM) dùng quản lí các thiết bị lưu trữ .LVM là một tiện ích cho phép chia không gian đĩa cứng thành những Logical Volume từ đó giúp cho việc thay đổi kích thước trở nên dễ dàng.
- Ưu điểm của LVM là tăng tính linh hoạt và khả năng kiểm soát. Các tập có thể được thay đổi kích thước động khi yêu cầu không gian thay đổi và di chuyển giữa các thiết bị vật lý trong nhóm trên hệ thống đang chạy hoặc xuất dễ dàng.
- LVM cũng cung cấp các tính năng nâng cao như Backup hệ thống bằng cách snapshot các phân vùng ổ cứng(real-time), Migrate dữ liệu dễ dàng.
- Physical Volume (PV): Ổ cứng vật lý (đĩa cứng, partition, SSD…) có thể chia thành nhiều phân vùng vật lý.
- Volume Group(VG): Là một nhóm bao gồm các Physical Volume.
- Logical Volume (Lv): Một Volume Group được chia nhỏ thành nhiều Logical Volume. Nó được dùng cho các để mount tới hệ thống tập tin (File System) và được format với những chuẩn định dạng khác nhau như ext2, ext3, ext4…

# 2.Tạo LVM
- Chạy lệnh `lvmdiskscan` để kiểm tra những thiết bị nào mà LVM có thể tương tác
  <img src="https://i.imgur.com/EgFXOCN.png">
  
- Tạo Physical Volume: `pvcreate /dev/tenphanvung(sda,sdb,...)` 
- Kiểm tra phân vùng PS: `pvs`
    <img src="https://i.imgur.com/1NzIVat.png">
    
- Thêm 1 Physical Volume vào Volume Group: 
    - `vgcreate testlvm(tenPS) /dev/sdb(Phan vung PV)histo`
- Kiểm tra Volume Group đã được tạo: `vgs`
    <img src="https://i.imgur.com/xgu9Gue.png">
    
- Tiếp theo ta sẽ tạo Logical Volume từ Volume Group:
    - `lvcreate -L(dung luong) size(dùng % hoặc mb,gb) -n(name) ten_logical_volume ten_groupvolume(vua tao o tren)`
    - `lvcreate -L 5G -n demo testlvm`
    - `lvcreate -L 5G -n demo1 testlvm`
    - `lvcreate -L 3G -n demo2 testlvm`
    
    <img src="https://i.imgur.com/sRpp6la.png">
    
- Sau khi tạo xong Logical Volume, ta sẽ định dạng LV bằng `mkfs`
  - `mkfs.ext4 /dev/testlvm/demo`
  - `mkfs.ext4 /dev/testlvm/demo1`
  - `mkfs.ext4 /dev/testlvm/demo2`
- Cuối cùng, chúng ta cần `mount` để sử dụng
  - Tạo thư mục để mount: `mkdir /mnt/{test,test1,test2}`
  - `mount /dev/testlvm/demo /mnt/test`
  - `mount /dev/testlvm/demo1 /mnt/test1`
  - `mount /dev/testlvm/demo2 /mnt/test2`
  - Hoặc sửa file `/etc/fstab` để mount tự động mỗi khi reboot
  
  
# 3. Thay đổi dung lượng
## 3.1 Mở rộng Volume Group và thay đổi kích thước các Logical Volume
- Sử dụng lệnh sau để thêm `/dev/sdc` vào Volume Group testlvm:
  - `vgextend testlvm /dev/sdc`
- Sử dụng lệnh `vgs` để kiểm tra dung lượng sau khi thêm.
- Để tăng kích thước Logical Volume:
  - `lvextend -L +10G /dev/testlvm/demo`
- Chạy lại lệnh `lvs` để kiểm tra dung lượng LV
    <img src="https://i.imgur.com/oBlEU2O.png">
    
- Sau khi tăng kích thước, ta dùng lệnh sau để resize:
  - `resize2fs /dev/testlvm/demo`
  
## 3.2 Giảm Logical Volume
- Khi muốn giảm dung lượng các Logical Volume, ta cần phải chú ý vì nó rất quan trọng và có thể bị lỗi. Để đảm bảo an toàn khi giảm Logical Volume cần thực hiện các bước sau:
  - Trước khi bắt đầu, cần sao lưu dữ liệu vì vậy sẽ không phải đau đầu nếu có sự cố xảy ra.
  - Để giảm dung lượng Logical Volume, cần thực hiện đầy đủ các bước sau:
    - Unmount file system.
    - Kiểm tra file system sau khi ngắt kết nối.
    - Giảm file system.
    - Giảm kích thước Logical Volume hơn kích thước hiện tại.
    - Kiểm tra lỗi cho file system.
    - Mount lại file system và kiểm tra kích thước của nó.
  
  - Unmount filesystem
    - `umount /mnt/test`
  - Kiểm tra lỗi file system bằng lệnh `e2fsck`
    - `e2fsck -f(force check) /dev/testlvm/demo`
  - Giảm kích thước của `demo` theo kích thước mong muốn.
    - `resize2fs /dev/testlvm/demo 10G`
    - `lvreduce -L 10G /dev/testlvm/demo`
  - Kiểm tra lại lỗi:
    - `e2fsck -f(force check) /dev/testlvm/demo`
  - Mount để sử dụng
    - `mount /dev/lvmtest/demo /mnt/test`
    - Sửa file `/etc/fstab` để mount cố định khi reboot
## 3.3 Xóa PV, VG, LV
- Để xóa LV ta cần `umount`: `umount /mnt/test`
- Xóa LV: `lvremove /dev/testlvm/demo`
- Xóa VG: `vgremove /dev/testlvm`
- Xóa PV: `pvremove /dev/sdb`
  
