- Add disk card raid0: 
    
    - Khi add disk SATA thì chỉ cần để chế độ AWB, Mode là WriteBack,Read ahead, Direct IO: `for i in {a..b};do storcli64 /cx add vd  type=r0 drives=8:$i AWB;done`. Set pdcache=off: `for i in {a..b};do storcli64 /cx/v$i set rdcache=ra wrcache=AWB pdcache=off; done`

    - Khi add disk SSD cần tuning một số tham số như sau: Mode là WriteThrough, No read ahead, Direct IO: `for i in {a..b};do storcli64 /cx add vd  type=r0 drives=8:$i WT;done`

    
    - Tắt cảnh báo ceph osd và ceph health (icinga)


- Tạo key: `ceph auth get-or-create client.bootstrap-osd | tee /var/lib/ceph/bootstrap-osd/ceph.keyring`

- Phân quyền: `chown ceph. /var/lib/ceph/bootstrap-osd/ceph.keyring`

- Zap disk: `for i in {a..b};do ceph-volume lvm zap /dev/sd$i --destroy;done`

- Prepare disk: `for i in {a..b};do ceph-volume lvm prepare --data /dev/sd$i ;done`

- Add osd vào crush rule (chỉnh lại tham số weight, root, host), xem osd nào đã được add rồi thì k add lại nữa, sử dụng awk để bắt osd, check lại trước khi chạy for: 
    
    - ``for i in `df -h | grep ceph | awk -F - {'print $2'}` ;do ceph osd crush add osd.$i 10.91359 root=Spunk host=bat-ceph-splunk-05 ;done``

- Active chạy vòng for giống như bước add osd vào crush rule(kiểm tra vòng for)

    - ``for i in `df -h | grep ceph | awk -F - {'print $2'}`;do ceph-volume lvm activate $i `cat /var/lib/ceph/osd/ceph-$i/fsid`; sleep 5 ;done``

- Nếu muốn xóa OSD (kiểm tra vòng for trước khi chạy): ``for i in `df -h| grep ceph | awk -F - {'print $2'}`;do ceph osd out ${i};ceph osd crush remove osd.${i};systemctl stop ceph-osd@${i};sleep 5;umount /var/lib/ceph/osd/ceph-${i};ceph auth del osd.${i};ceph osd rm ${i} ;done``

- Để tăng nhanh tốc độ backfill sang host mới => chỉnh backfill = 2. VD: add thêm bat-ceph-splunk-05, tìm tất cả osd thuộc host bat-ceph-splunk-01,02,03 => các host này đều thuộc root=Splunk

    - `ceph osd df tree > /home/phongdh/osd.txt`
    - `shift + V` xong di chuyển tới những dòng cần xóa => bấm `d` để xóa những cụm không cần tăng backfill
    - `cat /home/phongdh/osd.txt | grep osd | awk {'print $21'}` để in ra những osd trong cụm đó
    - ``for i in `cat /home/phongdh/osd.txt | grep osd | awk {'print $21'}`; do ceph tell $i injectargs '--osd-max-backfills 2' ;done``

    - Sau khi xong thì cần chuyển backfill về 1: ``for i in `cat /home/phongdh/osd.txt | grep osd | awk {'print $21'}`; do ceph tell $i injectargs '--osd-max-backfills 1' ;done``
