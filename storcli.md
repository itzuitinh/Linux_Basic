# Storcli là gì:
- Storcli là một công cụ thao tác trên command line nhằm mục đích quản lý raid hardware

# 1. Các câu lệnh thường dùng
- Show thông tin cơ bản của controller x :`storcli64 /cx show`

- Show tất cả thông tin của controller x: `storcli64 /cx show all`

- Show thông tin tất cả drives: `storcli64 /cx/dall show`

- Show thông tin raid:
` storcli64 /cx/vx show`

- Add physical drives thành virtual drives dùng raid: 

    - `storcli64 /cx add vd type=r(0,1,5,6) drives=EID:Slt(EID:Slt,Slt | EID:Slt-Slt) WB|WT`



- Set các trạng thái của virtual drives: `storcli64 /cx/vx set rdcache=ra|nora wrcache=AWB|WT pdcache=off`


- Show thông tin tất cả Virtual drives: `storcli64 /cx/vall show`

    - Cột cache:

        - NR|R: no read ahead | read ạhead 
        - WT|WB: WriteThrough | Write Back
        - D : DirectIO

- Delete virtual drives: `storcli64 /vx/vx|vall del`

- Xóa virtual drives khi có data `storcli64 /vx/vx|vall del force`

# 2. Các trạng thái của disk
- Ubad: trạng thái này thường gặp khi nhổ disk từ card raid này sang card raid khác

    - Chuyển trạng thái của disk từ Ubad sang UGood: `storcli64 /c1/ex/sx set good`

- Ugood: Khi ở trạng thái UGood mà disk group là F (foreign configuration: đã được setup ở card controller raid khác) có thể import disk vào 
hoặc clear foreign configuration sau đó add disk từ đầu

    - TH1 : có thể import vào raid nếu được:
    
        - `storcli64 /c1 /fall show`
        - `storcli64 /c1 /fall import`

    - TH2: xóa foreign configuration

        - `storcli64 /c1 /fall del`
        - Sau đó add disk như bình thường

- Controller bị cache offline hoặc mising virtual disks: `storcli64 /cx/vall delete preservedCache`

# 3.Locate disk
- Kiểm tra card raid có bật locate: `storcli64 /cx show activityforlocate`

- Nếu chưa thì cần enable locate: `storcli64 /cx set activityforlocate=on`

- Locate tất cả các disk: `for i in {0..10};do storcli64 /cx/eID/s${i} start locate;done`

- Stop locate disk cần tìm: `storcli64 /cx/eID/sx stop locate`

- Sau khi locate tất cả disk thì cần stop locate: `for i in {0..10};do storcli64 /cx/eID/s${i} stop locate;done`
