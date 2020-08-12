# 1.RAID Performance 
- RAID controller có thể cấu hình cách sử dụng cache. Tùy vào cấu hình mà nó ảnh hưởng đến hiệu năng khác nhau

- Read ahead: Nó sẽ tăng tốc độ đọc đối với kiểu truy cập dữ liệu tuần tự(HDD), nhưng khi truy cập dữ liệu ngẫu nhiên(SSD) thì không có tác dụng => HDD config mode Read ahead, SSD config no read ahead

- Write through: Dữ liệu sẽ được ghi trực tiếp lên Cache, cùng thời điểm này dữ liệu cũng sẽ được ghi vào disk => Ưu điểm: đam bảo dữ liệu luôn được an toàn trong trường hợp bị cúp điện, lỗi đột ngột

- Write back: Dữ liệu sẽ được ghi trực tiếp vào trong Cache, sau 1 thời gian chỉ định thì mới ghi vào disk => Ưu điểm là nhanh, nhưng không an toàn trong trường hợp lỗi đột ngột

- Khi add disk HDD thì nên dùng những option:

    - Mode là WriteBack
    - Read ahead
    - Direct IO

 - Khi add disk SDD thì nên dùng những option:

    - Mode là WriteThrough
    - No read ahead
    - Direct IO 

    
# 2. Benchmark

## 2.1 FIO

- Sequential Reads: `fio --name=test_seq_read --rw=read --direct=1 --ioengine=libaio --bs=8k --numjobs=8 --size=1G --runtime=600  --group_reporting`

- Sequential Writes: `fio --name=test_seq_write --rw=write --direct=1 --ioengine=libaio --bs=32k --numjobs=4 --size=2G --runtime=600 --group_reporting`

- Random Reads: `fio --name=test_rand_read --rw=randread --direct=1 --ioengine=libaio --bs=8k --numjobs=16 --size=1G --runtime=600 --group_reporting`

- Random Writes: `fio --name=test_rand_write --rw=randwrite --direct=1 --ioengine=libaio --bs=64k --numjobs=8 --size=512m --runtime=600 --group_reporting`

- Random Read/Writes (90% read, 10% write): `fio --name=test_rand_rw --rw=randrw --direct=1 --ioengine=libaio --bs=16k --numjobs=8 --rwmixread=90 --size=1G --runtime=600 --group_reporting`

- Các tùy chọn:

    - --name: tên file được tạo ra để test
    - --rw: phương thức đọc ghi(seq hoặc random)
    - --bs: block size
    - --numjobs: số lượng file
    - --size: kích thước file
    - --rwmixread: tỉ lệ read/write(90 => 90% read, 10% write)
    - --runtime: thời gian chạy benchmark (đơn vị: s)
    

## 2.2 Sysbench

- Sử dụng lệnh help: `sysbench fileio help`

- Tạo file để test: `sysbench fileio --file-num=8 --file-total-size=1G --file-test-mode=rndrw --time=300 --max-requests=0 prepare`

- Chạy benchmark: `sysbench fileio --file-num=8 --file-total-size=1G --file-test-mode=rndrw --time=300 --max-requests=0 run`

- Xóa file sau khi test xong: `sysbench fileio --file-num=8 --file-total-size=1G --file-test-mode=rndrw --time=300 --max-requests=0 cleanup`
