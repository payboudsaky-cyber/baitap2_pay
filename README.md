# baitap2_pay
Phần mở đầu: Thông tin cá nhân

    Họ và tên: kouson Mosaky

    Mã sinh viên: LAOS195089

    Lớp: K56KMT.K01

    Tên dự án: Quản lý Thư viện Sách (Library Management System)

    Phần 1: Khởi tạo bảng (Kiến thức 6, 7)
1.1. Mô tả logic hệ thống

Để quản lý việc mượn trả sách hiệu quả, hệ thống được thiết kế với 3 bảng dữ liệu chính có mối quan hệ chặt chẽ với nhau:

  Bảng Sach (Sách): Lưu trữ thông tin về các đầu sách có trong thư viện bao gồm mã sách, tên sách và số lượng hiện có trong kho.

  Bảng DocGia (Độc giả): Lưu trữ thông tin cá nhân của người mượn sách như họ tên và địa chỉ email để liên lạc.

  Bảng MuonSach (Mượn sách): Đây là bảng trung gian dùng để lưu vết các giao dịch mượn và trả sách. Bảng này liên kết với hai bảng trên thông qua các khóa ngoại (MaSach, MaDG) và theo dõi trạng thái của mỗi lần mượn.

1.2. Mã SQL khởi tạo dữ liệu

-- Tạo cơ sở dữ liệu mới mang mã số sinh viên LAOS195089
    CREATE DATABASE [QuanLyThuVien_LAOS195089];
    GO
    USE [QuanLyThuVien_LAOS195089];
    GO

-- 1. Tạo bảng Sách
CREATE TABLE [Sach] (
    [MaSach] INT PRIMARY KEY IDENTITY(1,1),
    [TenSach] NVARCHAR(200) NOT NULL,
    [SoLuong] INT CHECK ([SoLuong] >= 0)
);

-- 2. Tạo bảng Độc giả
    CREATE TABLE [DocGia] (
    [MaDG] INT PRIMARY KEY IDENTITY(1,1),
    [HoTen] NVARCHAR(100) NOT NULL,
    [Email] VARCHAR(100)
    );

-- 3. Tạo bảng Mượn sách
    CREATE TABLE [MuonSach] (
    [MaMuon] INT PRIMARY KEY IDENTITY(1,1),
    [MaSach] INT FOREIGN KEY REFERENCES [Sach]([MaSach]),
    [MaDG] INT FOREIGN KEY REFERENCES [DocGia]([MaDG]),
    [NgayMuon] DATETIME DEFAULT GETDATE(),
    [TrangThai] NVARCHAR(50) DEFAULT N'Đang mượn' -- Các trạng thái: Đang mượn, Đã trả
);

1.3. Hình ảnh minh chứng kết quả

<img width="960" height="540" alt="{5685A1E5-FC6D-4A67-938E-92D9CDD3AE20}" src="https://github.com/user-attachments/assets/87509045-f64c-42c4-9872-9a35020429a5" />

Chú thích: Ảnh chụp màn hình cho thấy em đã khởi tạo thành công cơ sở dữ liệu và 3 bảng dữ liệu chính. Các bảng đã được thiết lập khóa chính (Primary Key), khóa ngoại (Foreign Key) và các ràng buộc dữ liệu (Check Constraint) đầy đủ để đảm bảo tính toàn vẹn của hệ thống.

Phần 2: Xây dựng Function (Kiến thức 8, 9)
2.1. Tìm hiểu về Hàm có sẵn (Built-in Functions)

Trong SQL Server, các hàm Built-in được chia thành nhiều loại như: Hàm toán học, Hàm chuỗi, Hàm ngày tháng, Hàm hệ thống và Hàm tổng hợp (Aggregate).

Một số hàm hệ thống đặc sắc:

  GETDATE(): Trả về thời gian hiện tại của hệ thống.

  ISNULL(check_expression, replacement_value): Thay thế giá trị NULL bằng một giá trị chỉ định, rất hữu ích trong việc tránh lỗi tính toán.

DATEDIFF(interval, start, end): Tính toán khoảng cách giữa hai mốc thời gian.

 SQL khai thác:

    SELECT GETDATE() AS ThoiGianHienTai, 
    ISNULL(NULL, 0) AS XuLyNull,
    DATEDIFF(day, '2024-01-01', GETDATE()) AS SoNgayChenhLech;

<img width="960" height="540" alt="{956FEB6D-86AE-4716-8061-875DFDA2DF6B}" src="https://github.com/user-attachments/assets/92c46791-1626-4e6a-93ae-94ab8951dfef" />


2.2. Hàm do người dùng tự định nghĩa (User-Defined Functions - UDF)

Mục đích: Dùng để đóng gói các logic nghiệp vụ lặp đi lặp lại, giúp mã nguồn sạch sẽ, dễ bảo trì và tái sử dụng.

Tại sao cần tự viết: Mặc dù System Function rất nhiều nhưng không thể bao quát hết các logic nghiệp vụ riêng biệt của từng dự án (ví dụ: công thức tính lương riêng của một công ty, hoặc logic phân loại độc giả của một thư viện cụ thể).

Các loại hàm:

Scalar Function: Trả về một giá trị đơn duy nhất (số, chuỗi, ngày). Dùng khi cần tính toán nhanh một giá trị.

Inline Table-Valued Function: Trả về một bảng dữ liệu dựa trên một câu lệnh SELECT duy nhất. Hiệu suất cao, dùng như một khung nhìn (View) có tham số.

        Multi-statement Table-Valued Function: Trả về một bảng nhưng bên trong chứa nhiều câu lệnh xử lý phức tạp. Dùng khi cần lọc, tính toán qua nhiều bước trước khi xuất dữ liệu.

2.3. Thực hành viết các loại Function
A. Scalar Function (Hàm trả về một giá trị)

    Yêu cầu: Tính số ngày mà độc giả đã mượn sách tính đến thời điểm hiện tại.

    Mã SQL:

SQL

CREATE FUNCTION fn_TinhSoNgayMuon(@NgayMuon DATETIME)
RETURNS INT AS
BEGIN
    RETURN DATEDIFF(day, @NgayMuon, GETDATE());
END;

    Khai thác:

SQL

SELECT d.HoTen, m.MaSach, dbo.fn_TinhSoNgayMuon(m.NgayMuon) AS SoNgayMuon 
FROM DocGia d 
JOIN MuonSach m ON d.MaDG = m.MaDG;

<img width="960" height="540" alt="{675DBC53-D4A5-43C0-9A24-9F60DD9E5786}" src="https://github.com/user-attachments/assets/5a35294b-34de-4247-887c-25841fe17815" />


B. Inline Table-Valued Function (Hàm trả về bảng đơn giản)

    Yêu cầu: Tìm danh sách các sách còn trong kho theo số lượng tối thiểu do người dùng nhập vào.

    Mã SQL:

SQL

CREATE FUNCTION fn_TimSachTheoSoLuong(@MinQty INT)
RETURNS TABLE AS
RETURN (
    SELECT MaSach, TenSach, SoLuong 
    FROM Sach 
    WHERE SoLuong >= @MinQty
);

    Khai thác:

SQL

SELECT * FROM dbo.fn_TimSachTheoSoLuong(5); -- Tìm sách có từ 5 cuốn trở lên

<img width="960" height="540" alt="{E63E570D-210C-4F89-BDC3-22F57E53454A}" src="https://github.com/user-attachments/assets/639faef9-9c8f-4da0-88b3-a38075fb7417" />


C. Multi-statement Table-Valued Function (Hàm xử lý logic phức tạp)

    Yêu cầu: Phân loại độc giả dựa trên số ngày mượn sách. Nếu mượn trên 14 ngày là "Quá hạn", còn lại là "Trong hạn".

    Mã SQL:

CREATE FUNCTION fn_PhanLoaiMuonSach()
RETURNS @BangPhanLoai TABLE (
    MaDG INT,
    HoTen NVARCHAR(100),
    SoNgayMuon INT,
    TinhTrang NVARCHAR(50)
)
AS
BEGIN
    INSERT INTO @BangPhanLoai
    SELECT d.MaDG, d.HoTen, dbo.fn_TinhSoNgayMuon(m.NgayMuon),
           CASE 
               WHEN dbo.fn_TinhSoNgayMuon(m.NgayMuon) > 14 THEN N'Quá hạn'
               ELSE N'Trong hạn'
           END
    FROM DocGia d JOIN MuonSach m ON d.MaDG = m.MaDG;
    
    RETURN;
END;

    Khai thác:

SQL

SELECT * FROM dbo.fn_PhanLoaiMuonSach();

<img width="960" height="540" alt="{877995B6-9D17-4FF7-8234-488D0CBC5B41}" src="https://github.com/user-attachments/assets/05b3021b-4f76-4561-bd71-3d4b9a47b20a" />

Phần 3: Xây dựng Store Procedure (Kiến thức 10)
3.1. Tìm hiểu về Stored Procedure có sẵn (System Stored Procedures)

Trong SQL Server, các System Stored Procedures thường bắt đầu bằng tiền tố sp_ và được lưu trữ trong Database master. Chúng giúp người quản trị thực hiện các tác vụ quản lý hệ thống một cách nhanh chóng.

    Một số System SP đặc sắc:

        sp_help: Cung cấp thông tin chi tiết về bất kỳ đối tượng nào trong database (bảng, view, index...).

        sp_rename: Dùng để đổi tên một đối tượng (như đổi tên bảng hoặc tên cột).

        sp_helpdb: Liệt kê thông tin về các cơ sở dữ liệu có trong máy chủ.

    Cách dùng:

SQL

EXEC sp_help 'Sach'; -- Xem cấu trúc của bảng Sach
EXEC sp_helpdb;      -- Xem danh sách các database hiện có

3.2. Thực hành viết Stored Procedure
A. Store Procedure thực hiện lệnh INSERT/UPDATE có kiểm tra logic

    Yêu cầu: Tạo thủ tục để thêm sách mới. Trước khi thêm, phải kiểm tra xem tên sách đã tồn tại chưa. Nếu tồn tại rồi thì chỉ cập nhật số lượng cộng thêm, nếu chưa có thì mới INSERT mới.

    Mã SQL:

SQL

CREATE PROCEDURE sp_NhapSachMoi
    @TenSach NVARCHAR(200),
    @SoLuong INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Sach WHERE TenSach = @TenSach)
    BEGIN
        -- Nếu sách đã có, cập nhật thêm số lượng
        UPDATE Sach SET SoLuong = SoLuong + @SoLuong WHERE TenSach = @TenSach;
        PRINT N'Sách đã tồn tại, đã cập nhật thêm số lượng.';
    END
    ELSE
    BEGIN
        -- Nếu chưa có, thêm mới hoàn toàn
        INSERT INTO Sach (TenSach, SoLuong) VALUES (@TenSach, @SoLuong);
        PRINT N'Đã thêm sách mới thành công.';
    END
END;

    Khai thác:

SQL

EXEC sp_NhapSachMoi N'Lập trình SQL cơ bản', 10;

<img width="960" height="540" alt="{3ECE641F-A5E4-4F2D-A215-0952F458F126}" src="https://github.com/user-attachments/assets/d8fe24ed-dddd-438f-9a78-76c6ed13ab24" />


B. Store Procedure sử dụng tham số OUTPUT

    Yêu cầu: Tính tổng số lượng sách đang có trong thư viện và trả giá trị đó về một biến để sử dụng tiếp.

    Mã SQL:

SQL

CREATE PROCEDURE sp_TongSoLuongSach
    @TongSach INT OUTPUT
AS
BEGIN
    SELECT @TongSach = SUM(SoLuong) FROM Sach;
END;

    Khai thác:

SQL

DECLARE @Result INT;
EXEC sp_TongSoLuongSach @TongSach = @Result OUTPUT;
PRINT N'Tổng số sách trong kho là: ' + CAST(@Result AS VARCHAR);

<img width="960" height="540" alt="{2E18340A-F16D-4D4E-BCED-01A0D534746E}" src="https://github.com/user-attachments/assets/a09d873d-ce08-49e4-b608-389f0f814812" />


C. Store Procedure trả về một tập kết quả (Result set) từ JOIN nhiều bảng

    Yêu cầu: Xuất danh sách chi tiết các đơn mượn sách bao gồm: Tên độc giả, Tên sách và Ngày mượn.

    Mã SQL:

    CREATE PROCEDURE sp_DanhSachChiTietMuonSach
    AS
    BEGIN
    SELECT d.HoTen AS [Tên Độc Giả], 
           s.TenSach AS [Tên Sách], 
           m.NgayMuon AS [Ngày Mượn],
           m.TrangThai AS [Trạng Thái]
    FROM MuonSach m
    JOIN DocGia d ON m.MaDG = d.MaDG
    JOIN Sach s ON m.MaSach = s.MaSach;
    END;
    
    --Khai thác:
     
    EXEC sp_DanhSachChiTietMuonSach;

<img width="960" height="540" alt="{6FE43B12-575A-4A57-923C-9FA92283785F}" src="https://github.com/user-attachments/assets/97742fae-4c99-4f2c-b463-0e83efdfac11" />

Phần 4: Trigger và Xử lý logic nghiệp vụ (Kiến thức 11)
4.1. Xây dựng Trigger tự động hóa nghiệp vụ

Logic nghiệp vụ: Khi có một bản ghi mới được thêm vào bảng MuonSach (Độc giả mượn sách), hệ thống phải tự động giảm số lượng sách tương ứng trong kho (bảng Sach) để đảm bảo dữ liệu tồn kho luôn chính xác.

Mã SQL tạo Trigger:
SQL

USE [QuanLyThuVien_LAOS195089];
GO

CREATE TRIGGER trg_MuonSach_UpdateKho
ON MuonSach
AFTER INSERT
AS
BEGIN
    -- Cập nhật giảm số lượng sách trong kho khi có người mượn
    UPDATE Sach
    SET SoLuong = SoLuong - 1
    FROM Sach s 
    JOIN inserted i ON s.MaSach = i.MaSach;
    
    PRINT N'Trigger đã tự động cập nhật giảm số lượng sách trong kho.';
END;
GO

    Khai thác Trigger: Thực hiện lệnh INSERT một đơn mượn sách mới và kiểm tra bảng Sach.

SQL

-- Giả sử MaSach = 1 đang có 10 cuốn, sau lệnh này sẽ còn 9 cuốn
INSERT INTO MuonSach (MaSach, MaDG, NgayMuon, TrangThai) 
VALUES (1, 1, GETDATE(), N'Đang mượn');

SELECT * FROM Sach WHERE MaSach = 1;

<img width="960" height="540" alt="{21D5D7D0-3917-4A31-895D-8D647FE71457}" src="https://github.com/user-attachments/assets/5a912126-4484-4dce-95a1-46e498b41c65" />


Phần 4: Trigger và Xử lý logic nghiệp vụ (Kiến thức 11)
4.1. Mục đích và ý nghĩa của Trigger

Trong hệ thống quản lý thư viện, việc cập nhật số lượng sách trong kho một cách thủ công mỗi khi có người mượn hoặc trả sách rất dễ gây ra sai sót dữ liệu. Do đó, em xây dựng Trigger để tự động hóa quy trình này, đảm bảo tính nhất quán giữa bảng mượn sách (MuonSach) và bảng kho sách (Sach).
4.2. Xây dựng Trigger

Dựa trên mã nguồn em đã thực hiện trong hình image_15.png, Trigger này sẽ tự động cập nhật số lượng sách ngay sau khi có thao tác mượn hoặc trả sách diễn ra.

Mã SQL thực hiện:
SQL

USE [QuanLyThuVien_LAOS195089];
GO

CREATE TRIGGER trg_CapNhatKhoSach
ON MuonSach
AFTER INSERT, UPDATE
AS
BEGIN
    -- Nếu là mượn sách mới: Trừ số lượng trong kho
    IF EXISTS (SELECT * FROM inserted WHERE TrangThai = N'Đang mượn')
    BEGIN
        UPDATE Sach
        SET SoLuong = SoLuong - 1
        FROM Sach s JOIN inserted i ON s.MaSach = i.MaSach;
        PRINT N'Trigger: Đã trừ 1 sách trong kho.';
    END

    -- Nếu là trả sách: Cộng lại số lượng vào kho
    IF EXISTS (SELECT * FROM inserted i JOIN deleted d ON i.MaMuon = d.MaMuon 
               WHERE i.TrangThai = N'Đã trả' AND d.TrangThai = N'Đang mượn')
    BEGIN
        UPDATE Sach
        SET SoLuong = SoLuong + 1
        FROM Sach s JOIN inserted i ON s.MaSach = i.MaSach;
        PRINT N'Trigger: Đã cộng lại 1 sách vào kho.';
    END
END;

4.3. Kiểm tra và khai thác Trigger

Để minh chứng cho sự hoạt động của Trigger, em thực hiện lệnh UPDATE trạng thái mượn sách và kiểm tra bảng Sach.

Câu lệnh kiểm tra:
SQL

-- Giả sử độc giả trả sách, cập nhật trạng thái
UPDATE MuonSach SET TrangThai = N'Đã trả' WHERE MaMuon = 1;

-- Kiểm tra số lượng sách tự động tăng lên
SELECT * FROM Sach;

4.4. Hình ảnh minh chứng kết quả

<img width="960" height="540" alt="{21D5D7D0-3917-4A31-895D-8D647FE71457}" src="https://github.com/user-attachments/assets/5a912126-4484-4dce-95a1-46e498b41c65" />


<img width="960" height="540" alt="{F1DCD143-21C0-4396-A291-32409034BE4E}" src="https://github.com/user-attachments/assets/9be40d52-fad4-44dd-be9c-f7681e897fe2" />

Chú thích: Qua hình ảnh, có thể thấy Trigger đã hoạt động chính xác theo logic nghiệp vụ. Khi trạng thái mượn trả thay đổi, số lượng tồn kho của sách cũng tự động thay đổi tương ứng mà không cần can thiệp thủ công.

Phần 5: Cursor và Duyệt dữ liệu (Kiến thức 11)
5.1. Sử dụng CURSOR để xử lý dữ liệu theo từng dòng

Logic nghiệp vụ: Hệ thống cần duyệt qua danh sách các độc giả đang mượn sách. Nếu số ngày mượn (tính từ hàm fn_TinhSoNgayMuon) lớn hơn 14 ngày, hệ thống sẽ in ra cảnh báo "Yêu cầu thu hồi sách". Nếu dưới 14 ngày, in ra "Vẫn trong hạn".

Mã SQL thực hiện Cursor:
SQL

USE [QuanLyThuVien_LAOS195089];
GO

-- Khai báo các biến để chứa dữ liệu từ Cursor
DECLARE @TenDocGia NVARCHAR(100);
DECLARE @NgayMuon DATETIME;
DECLARE @SoNgayDaMuon INT;

-- 1. Khai báo Cursor
DECLARE cur_KiemTraHanMuon CURSOR FOR 
SELECT d.HoTen, m.NgayMuon 
FROM DocGia d JOIN MuonSach m ON d.MaDG = m.MaDG
WHERE m.TrangThai = N'Đang mượn';

-- 2. Mở Cursor
OPEN cur_KiemTraHanMuon;

-- 3. Duyệt qua từng bản ghi
FETCH NEXT FROM cur_KiemTraHanMuon INTO @TenDocGia, @NgayMuon;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @SoNgayDaMuon = dbo.fn_TinhSoNgayMuon(@NgayMuon);
    
    IF @SoNgayDaMuon > 14
        PRINT N'Độc giả: ' + @TenDocGia + N' - Cảnh báo: Yêu cầu thu hồi sách gấp!';
    ELSE
        PRINT N'Độc giả: ' + @TenDocGia + N' - Trạng thái: Vẫn trong hạn mượn.';

    -- Đọc bản ghi tiếp theo
    FETCH NEXT FROM cur_KiemTraHanMuon INTO @TenDocGia, @NgayMuon;
END;

-- 4. Đóng và giải phóng Cursor
CLOSE cur_KiemTraHanMuon;
DEALLOCATE cur_KiemTraHanMuon;

<img width="960" height="540" alt="{51736AA1-0B85-4F67-9FAE-CB62907332ED}" src="https://github.com/user-attachments/assets/12293ffa-740e-4650-ad8a-e03cf599a140" />


5.2. Giải quyết bài toán không dùng CURSOR

Trong SQL Server, chúng ta hoàn toàn có thể sử dụng câu lệnh SELECT kết hợp với CASE WHEN để đạt kết quả tương tự với hiệu suất cao hơn nhiều.

Mã SQL thay thế:
SQL

SELECT d.HoTen, 
       dbo.fn_TinhSoNgayMuon(m.NgayMuon) AS SoNgayMuon,
       CASE 
           WHEN dbo.fn_TinhSoNgayMuon(m.NgayMuon) > 14 THEN N'Yêu cầu thu hồi sách gấp!'
           ELSE N'Vẫn trong hạn mượn.'
       END AS ThongBaoCanhBao
FROM DocGia d JOIN MuonSach m ON d.MaDG = m.MaDG
WHERE m.TrangThai = N'Đang mượn';

<img width="960" height="540" alt="{82AA4A88-F154-48FE-83F6-2CC9BEE2A9FC}" src="https://github.com/user-attachments/assets/c277cfa4-d0f0-48b8-bdf1-90918d47508a" />


So sánh tốc độ:

    Có dùng Cursor: Xử lý chậm hơn vì SQL Server phải duyệt qua từng dòng một (Row-by-row), tốn nhiều tài nguyên CPU và bộ nhớ.

    Không dùng Cursor (Dùng SELECT): Nhanh hơn gấp nhiều lần vì SQL Server tối ưu hóa truy vấn trên toàn bộ tập dữ liệu (Set-based) cùng một lúc.

    Minh chứng: Đối với bảng dữ liệu lớn (hàng triệu dòng), Cursor có thể mất vài phút trong khi SELECT chỉ mất vài giây.

5.3. Trường hợp bắt buộc phải sử dụng CURSOR

Mặc dù SELECT thông thường rất mạnh, nhưng có những trường hợp Cursor là lựa chọn cần thiết hoặc dễ kiểm soát hơn, ví dụ:

    Thực thi các tác vụ quản trị hệ thống: Khi cần duyệt qua danh sách các Database để thực hiện Backup từng cái một, hoặc duyệt qua các Table để phân mảnh (Rebuild Index) từng Table.

    Gọi Store Procedure phức tạp cho từng dòng: Khi mỗi bản ghi cần được truyền vào một Store Procedure khác để xử lý logic bên ngoài SQL (như gửi Email thông báo riêng cho từng độc giả quá hạn). Câu lệnh SELECT thông thường khó có thể gọi Procedure cho từng dòng một cách linh hoạt như Cursor.
