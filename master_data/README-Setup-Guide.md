# 90S COFFEE – Odoo 17 Setup & Master Data Import Guide

Tài liệu hướng dẫn cài đặt môi trường Odoo 17 Community bằng Docker Compose, khởi tạo database, cấu hình các phân hệ và import dữ liệu mẫu cho dự án **90S COFFEE**.

Mục tiêu là giúp các thành viên trong nhóm có thể tự thiết lập môi trường Odoo trên máy tính cá nhân và sử dụng chung bộ dữ liệu ban đầu của dự án.

---

## 1. Technology Stack

| Thành phần | Phiên bản / Công nghệ |
|---|---|
| ERP | Odoo Community 17.0 |
| Database | PostgreSQL 15 |
| Container | Docker |
| Container Orchestration | Docker Compose |
| Operating System | Windows 11 |
| Custom Addons | Python, XML |
| Master Data | CSV (UTF-8) |

### 1.1. Yêu cầu môi trường

Trước khi bắt đầu, cần cài đặt:

- Docker Desktop.
- WSL 2 (đối với Windows).
- Git.
- Visual Studio Code (khuyến nghị).

Kiểm tra Docker đã hoạt động bằng PowerShell hoặc Terminal:

```bash
docker --version
docker compose version
git --version
```

Đảm bảo Docker Desktop đang chạy trước khi thực hiện các bước tiếp theo.

---

## 2. Clone Repository

Mở Terminal hoặc PowerShell và di chuyển đến thư mục muốn lưu dự án.

Clone repository:

```bash
git clone https://github.com/XaLach-Chu/IS336.R12-90S-Coffee-Custom-Addons.git
```

Di chuyển vào thư mục dự án:

```bash
cd IS336.R12-90S-Coffee-Custom-Addons
```

Cấu trúc thư mục chính:

```text
IS336.R12-90S-Coffee-Custom-Addons/
│
├── custom_addons/
│
├── master_data/
│   ├── README-Setup-Guide.md
│   ├── 00_contact_tags.csv
│   ├── 01_categories.csv
│   ├── 02_locations.csv
│   ├── 03_work_centers.csv
│   ├── 04_vendors.csv
│   ├── 05_products.csv
│   ├── 06_pricelists.csv
│   ├── 07_customers.csv
│   ├── 08_bom.csv
│   ├── 09_employees.csv
│   ├── 10_opening_stock.csv
│   ├── 11_reordering_rule.csv
│   ├── 12_purchase_orders.csv
│   └── 13_sales_orders.csv
│
├── api_scripts/
│
├── requirements.txt
│
└── docker-compose.yml
```

*Lưu ý: Tên file trong ví dụ có thể khác với tên thực tế trong repository. Cần giữ đúng tên file CSV đang được nhóm sử dụng.*

---

## 3. Khởi tạo môi trường bằng Docker Compose

### 3.1. Docker Compose là gì?

Docker Compose cho phép khởi chạy nhiều container cùng lúc bằng một file cấu hình.

Trong dự án này, hệ thống gồm hai container chính:

- **PostgreSQL 15:** Quản lý và lưu trữ dữ liệu của Odoo.
- **Odoo 17:** Cung cấp giao diện ERP và xử lý các nghiệp vụ quản lý.

Thay vì phải cài đặt PostgreSQL và Odoo trực tiếp trên Windows, Docker sử dụng các image tương ứng để khởi tạo môi trường riêng biệt.

### 3.2. Cấu hình docker-compose.yml

File `docker-compose.yml` được đặt tại thư mục gốc của repository.

Cấu hình tham khảo:

```yaml
services:
  db:
    image: postgres:15
    container_name: 90s-coffee-db
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: odoo
      POSTGRES_PASSWORD: odoo
    volumes:
      - db-data:/var/lib/postgresql/data
    restart: unless-stopped

  odoo:
    image: odoo:17.0
    container_name: 90s-coffee-odoo
    depends_on:
      - db
    ports:
      - "8069:8069"
    environment:
      HOST: db
      USER: odoo
      PASSWORD: odoo
    volumes:
      - odoo-data:/var/lib/odoo
      - ./custom_addons:/mnt/extra-addons
    restart: unless-stopped

volumes:
  db-data:
  odoo-data:
```

**Giải thích các thành phần:**

| Cấu hình | Ý nghĩa |
|---|---|
| `postgres:15` | Image PostgreSQL phiên bản 15 |
| `odoo:17.0` | Image Odoo phiên bản 17 |
| `depends_on` | Khai báo Odoo phụ thuộc vào PostgreSQL |
| `8069:8069` | Ánh xạ cổng truy cập Odoo từ máy tính |
| `db-data` | Lưu trữ dữ liệu PostgreSQL |
| `odoo-data` | Lưu trữ dữ liệu của Odoo |
| `./custom_addons` | Đồng bộ thư mục custom addons từ máy tính vào container |

**Lưu ý:** Các thông tin kết nối PostgreSQL trong file Compose phải thống nhất giữa hai service. Nếu repository đang sử dụng cấu hình khác hoặc file `.env`, hãy sử dụng cấu hình thực tế của repository thay vì ghi đè tùy tiện.

### 3.3. Khởi chạy hệ thống

Tại thư mục gốc dự án, chạy:

```bash
docker compose up -d
```

Docker sẽ tự động:

1. Tìm image PostgreSQL 15.
2. Tìm image Odoo 17.
3. Tải các image nếu máy tính chưa có.
4. Tạo container PostgreSQL.
5. Tạo container Odoo.
6. Khởi chạy hai container và kết nối Odoo với PostgreSQL.

Kiểm tra trạng thái:

```bash
docker compose ps
```

Nếu cả hai container đang chạy, môi trường đã được khởi tạo.

Xem log khi cần kiểm tra lỗi:

```bash
docker compose logs -f
```

### 3.4. Truy cập Odoo

Mở trình duyệt và truy cập:

```text
http://localhost:8069
```

Odoo sẽ hiển thị giao diện quản lý database.

---

## 4. Tạo Database

Tại màn hình Database Manager, chọn **Create Database**.

Nhập các thông tin:

| Trường | Giá trị |
|---|---|
| Master Password | Master password được cấu hình cho Database Manager |
| Database Name | `90s_coffee` |
| Email | Email tài khoản quản trị |
| Password | Mật khẩu tài khoản quản trị |
| Phone Number | Có thể để trống |
| Language | English hoặc Vietnamese |
| Country | Vietnam |
| Demo Data | Không chọn |

Nhấn **Create Database**.

Sau khi tạo thành công, Odoo sẽ chuyển đến giao diện quản trị hệ thống.

**Phân biệt hai loại mật khẩu:**

- Master Password: Dùng để quản lý database trên Database Manager.
- Email/Password: Dùng để đăng nhập vào tài khoản Odoo Administrator.

---

## 5. Cài đặt các ứng dụng Odoo

Truy cập ứng dụng **Apps** và cài đặt các phân hệ cần thiết cho dự án.

| Ứng dụng | Chức năng |
|---|---|
| Contacts | Quản lý khách hàng và nhà cung cấp |
| Sales | Quản lý báo giá và đơn bán hàng |
| Purchase | Quản lý yêu cầu báo giá và đơn mua hàng |
| Inventory | Quản lý kho và tồn kho |
| Manufacturing | Quản lý sản xuất và định mức nguyên vật liệu |
| Employees | Quản lý nhân viên |

Sau khi cài đặt, kiểm tra các ứng dụng đã xuất hiện trên giao diện chính.

---

## 6. Cấu hình hệ thống trước khi Import

Đây là bước quan trọng. Một số chức năng cần được bật trước khi nhập dữ liệu CSV.

### 6.1. Cấu hình Inventory

Truy cập:

`Inventory → Configuration → Settings`

Bật tính năng:

- Storage Locations: Cho phép quản lý tồn kho theo vị trí.

Nhấn **Save**.

### 6.2. Cấu hình Manufacturing

Truy cập:

`Manufacturing → Configuration → Settings`

Bật tính năng:

- Work Orders: Cho phép quản lý các công đoạn sản xuất tại Work Centers.

Nhấn **Save**.

### 6.3. Cấu hình Sales Pricelists

Truy cập:

`Sales → Configuration → Settings`

Trong phần Pricing, bật:

- Pricelists.

Nhấn **Save**.

Tính năng này cho phép sử dụng nhiều bảng giá khác nhau cho khách hàng bán lẻ, VIP, đại lý và OEM.

### 6.4. Kiểm tra đơn vị tính

Đảm bảo hệ thống có các đơn vị tính phù hợp:

- kg: Sử dụng cho nhân xanh và dịch vụ rang gia công.
- Units: Sử dụng cho sản phẩm đóng gói, bao bì và phụ liệu.

Đơn vị tính của sản phẩm phải phù hợp với dữ liệu trong CSV và BoM.

---

## 7. Quy trình Import Master Data

### 7.1. Nguyên tắc Import

Các file CSV cần được import theo thứ tự từ 01 đến 13.

Lý do: dữ liệu ở các bước sau có thể tham chiếu đến dữ liệu đã được tạo ở bước trước.

Ví dụ:

- Sản phẩm cần có Product Category.
- BoM cần có sản phẩm và nguyên vật liệu.
- Opening Stock cần có sản phẩm và Location.
- Purchase Orders cần có Vendor và Product.
- Sales Orders cần có Customer và Product.

**Không nên import tùy ý hoặc đảo thứ tự các file.**

### 7.2. Cách Import CSV trên Odoo

Quy trình chung:

1. Mở ứng dụng tương ứng.
2. Truy cập màn hình danh sách dữ liệu cần import.
3. Chọn biểu tượng thao tác hoặc menu phù hợp.
4. Chọn **Import Records**.
5. Tải file CSV từ thư mục `master_data`.
6. Kiểm tra các trường ánh xạ giữa CSV và Odoo.
7. Nhấn **Test** để kiểm tra dữ liệu.
8. Nếu không có lỗi, nhấn **Import**.
9. Kiểm tra dữ liệu đã xuất hiện trong danh sách.

Ví dụ:

```text
CSV Column                 Odoo Field
------------------------------------------
Product                    Product
Location                   Location
Counted Quantity           Counted Quantity
```

Một số trường quan hệ như Product, Vendor, Customer, Location hoặc BoM cần khớp với dữ liệu đã có trong Odoo.

**Lưu ý khi import:**

- Sử dụng CSV UTF-8.
- Kiểm tra tên sản phẩm và mã nội bộ.
- Kiểm tra định dạng ngày tháng.
- Kiểm tra số lượng và đơn giá.
- Không tự ý đổi tên các giá trị tham chiếu nếu chưa kiểm tra trong Odoo.
- Không import lại cùng một file nhiều lần nếu chưa kiểm tra dữ liệu đã tồn tại, tránh tạo bản ghi trùng lặp.

---

## 8. Danh sách các file Master Data
### 00 – Contact Tags

**Mục đích:** Khởi tạo các nhãn phân loại khách hàng và nhà cung cấp.

Truy cập:

`Contacts → Configuration → Contact Tags`

Import file:

`00_contact_tags.csv`

Các Contact Tags gồm:

- Nhà cung cấp nhân xanh
- Nhà cung cấp bao bì
- Nhà cung cấp phụ liệu
- VIP
- Quán cà phê
- Đại lý
- OEM
- HoReCa
- Kênh online

**Kiểm tra sau import:**

- Có đủ 9 Contact Tags.
- Tên nhãn hiển thị chính xác.
- Không có nhãn trùng lặp.

Các nhãn này được sử dụng khi import Vendors và Customers ở bước 04 và 07.

### 01 – Product Categories

**Mục đích:** Khởi tạo hệ thống danh mục sản phẩm.

Truy cập:

`Inventory → Configuration → Product Categories`

Các nhóm danh mục chính:

- All / Thành phẩm / Cà phê rang xay
- All / Nguyên vật liệu / Nhân xanh
- All / Nguyên vật liệu / Bao bì - Phụ liệu
- All / Quà tặng khuyến mãi
- All / Dịch vụ

Import file:

`01_categories.csv`

Kiểm tra sau import: Các danh mục và quan hệ cha – con hiển thị đúng cấu trúc.

### 02 – Locations

**Mục đích:** Khởi tạo các vị trí lưu kho.

Truy cập:

`Inventory → Configuration → Locations`

Các vị trí cần có:

- 90S/Stock/Kho Nhân xanh
- 90S/Stock/Kho Bao bì
- 90S/Stock/Xưởng sản xuất
- 90S/Stock/Kho Thành phẩm

Import file:

`02_locations.csv`

Kiểm tra sau import: Các vị trí được tạo dưới đúng kho 90S và có loại vị trí phù hợp.

### 03 – Work Centers

**Mục đích:** Khởi tạo các khu vực và thiết bị phục vụ sản xuất.

Truy cập:

`Manufacturing → Configuration → Work Centers`

Các Work Centers:

| Tên | Mã | Chi phí/giờ |
|---|---|---:|
| Máy rang cà phê | WC-RANG | 120.000 VND |
| Khu đóng gói | WC-DONGGOI | 60.000 VND |
| Bàn kiểm tra chất lượng (QC) | WC-QC | 50.000 VND |

Import file:

`03_work_centers.csv`

Kiểm tra sau import: Có đủ 3 Work Centers và thông tin chi phí hoạt động.

### 04 – Vendors

**Mục đích:** Khởi tạo danh sách nhà cung cấp.

Truy cập:

`Contacts → Contacts`

Import file:

`04_vendors.csv`

Các nhà cung cấp bao gồm:

1. Vựa Nhân Xanh Pleiku.
2. HTX Cà Phê Đắk Lắk.
3. HTX Cà Phê Cầu Đất Xanh.
4. Công ty TNHH Bao Bì Á Châu.
5. Công ty TNHH In Nhãn Việt.
6. Công ty TNHH Thủy Tinh Bình Minh.

Kiểm tra sau import:

- Tên nhà cung cấp.
- Địa chỉ.
- Số điện thoại.
- Email.
- Mã số thuế.
- Contact Tags.

### 05 – Products

**Mục đích:** Khởi tạo danh sách sản phẩm, nguyên vật liệu, bao bì, quà tặng và dịch vụ.

Truy cập:

`Inventory → Products → Products`

Import file:

`05_products.csv`

Dữ liệu gồm:

- Nhân xanh Robusta, Arabica, Culi, Moka và Honey.
- Túi cà phê 250g và 1kg.
- Tem nhãn 90S Coffee.
- Thùng carton.
- Ly thủy tinh 90S.
- Các dòng cà phê thành phẩm.
- Dịch vụ rang gia công và thiết kế bao bì.

Tổng số sản phẩm và dịch vụ: 24.

Kiểm tra sau import:

- Product Name.
- Internal Reference.
- Product Category.
- Unit of Measure.
- Cost.
- Sales Price.
- Product Type và các tùy chọn mua/bán phù hợp.

### 06 – Pricelists

**Mục đích:** Khởi tạo các bảng giá phục vụ hoạt động kinh doanh.

Truy cập:

`Sales → Products → Pricelists`

Import file:

`06_pricelists.csv`

Các bảng giá chính:

| Bảng giá | Chính sách |
|---|---|
| Giá bán lẻ | Giá bán mặc định |
| Giá VIP | Giảm 5% |
| Giá Đại lý | Giảm 10% cho cà phê thành phẩm theo điều kiện số lượng |
| Giá OEM - Rang gia công | Đơn giá theo sản lượng rang |

Đối với bảng giá OEM, cần kiểm tra các mức giá dịch vụ rang gia công:

| Sản lượng tối thiểu | Giá |
|---:|---:|
| 30 kg | 10.000 VND/kg |
| 90 kg | 8.000 VND/kg |
| 1.000 kg | 6.000 VND/kg |

Lưu ý: Pricelist Rules cần được kiểm tra sau import. Nếu file chỉ tạo bảng giá mà chưa tạo đầy đủ quy tắc giá, cần bổ sung các quy tắc trực tiếp trên Odoo.

### 07 – Customers

**Mục đích:** Khởi tạo danh sách khách hàng.

Truy cập:

`Contacts → Contacts`

Import file:

`07_customers.csv`

Kiểm tra sau import:

- Tên khách hàng.
- Loại liên hệ.
- Địa chỉ.
- Số điện thoại.
- Email.
- Mã số thuế.
- Contact Tags.

Các nhóm khách hàng có thể bao gồm:

- VIP.
- Quán cà phê.
- Đại lý.
- OEM.
- HoReCa.
- Kênh online.

Lưu ý: Contact Tags dùng để phân loại và tìm kiếm khách hàng. Muốn tự động áp dụng giá VIP hoặc giá Đại lý, cần cấu hình Pricelist tương ứng trên hồ sơ khách hàng.

### 08 – BoM (Bills of Materials)

**Mục đích:** Khởi tạo định mức nguyên vật liệu và công đoạn sản xuất cho từng thành phẩm.

Truy cập:

`Manufacturing → Products → Bills of Materials`

Import file:

`08_bom.csv`

Dữ liệu gồm 10 BoM cho các sản phẩm cà phê thành phẩm.

Nguyên tắc định mức:

- Mỗi mẻ tiêu chuẩn sử dụng 12 kg nhân xanh.
- Sau quá trình rang, thu được 10 kg thành phẩm.
- Tỷ lệ hao hụt khoảng 16,7%.
- Với sản phẩm 250g: 10 kg tương ứng 40 gói.
- Với sản phẩm 1kg: 10 kg tương ứng 10 gói.
- Mỗi gói sử dụng 1 túi và 1 tem nhãn.

Các công đoạn sản xuất:

| Công đoạn | Thời gian |
|---|---:|
| Rang cà phê | 30 phút |
| Đóng gói 250g | 20 phút |
| Đóng gói 1kg | 10 phút |
| Kiểm tra chất lượng (QC) | 10 phút |

Kiểm tra sau import:

- Sản phẩm thành phẩm.
- BoM Type.
- Components.
- Quantity.
- Operations.
- Work Centers.
- Manual Duration.

Lưu ý: Cần đảm bảo BoM liên kết đúng Work Centers đã tạo ở bước 03.

### 09 – Employees

**Mục đích:** Khởi tạo danh sách nhân viên và phòng ban.

Truy cập:

`Employees → Employees`

Import file:

`09_employees.csv`

Danh sách nhân viên gồm 7 người thuộc các bộ phận:

- Ban Giám đốc.
- Kinh doanh.
- Mua hàng.
- Kho.
- Sản xuất.
- Kế toán.

Kiểm tra sau import:

- Employee Name.
- Job Position.
- Department.
- Work Contact Information nếu có.

### 10 – Opening Stock

**Mục đích:** Khởi tạo số lượng tồn kho đầu kỳ tại ngày 01/07/2026.

Truy cập:

`Inventory → Operations → Physical Inventory`

Import file:

`10_opening_stock.csv`

File gồm 20 dòng dữ liệu với ba thông tin chính:

| CSV Column | Odoo Field |
|---|---|
| Location | Location |
| Product | Product |
| Counted Quantity (01/07/2026) | Counted Quantity |

Sau khi import:

1. Kiểm tra đủ 20 dòng.
2. Kiểm tra đúng Location.
3. Kiểm tra đúng Product.
4. Kiểm tra Counted Quantity.
5. Chọn **Apply All** để xác nhận điều chỉnh tồn kho.

**Quan trọng:** Sau khi import, On Hand có thể vẫn bằng 0 trong khi Counted Quantity đã có số liệu. Cần Apply để cập nhật tồn kho thực tế.

### 11 – Reordering Rules

**Mục đích:** Thiết lập quy tắc tái đặt hàng khi tồn kho xuống dưới mức tối thiểu.

Truy cập:

`Inventory → Operations → Replenishment`

Import file:

`11_reordering_rule.csv`

File gồm 9 quy tắc cho các sản phẩm nguyên vật liệu và bao bì.

Các trường chính:

- Product.
- Location.
- Minimum Quantity.
- Maximum Quantity.

Kiểm tra sau import:

- Đúng sản phẩm.
- Đúng vị trí lưu kho.
- Đúng số lượng tối thiểu (Min).
- Đúng số lượng tối đa (Max).

Khi tồn kho giảm xuống dưới Min, Odoo có thể đề xuất bổ sung hàng để đưa tồn kho về Max.

### 12 – Purchase Orders

**Mục đích:** Khởi tạo dữ liệu đơn mua hàng từ nhà cung cấp.

Truy cập:

`Purchase → Orders → Purchase Orders`

Import file:

`12_purchase_orders.csv`

Dữ liệu gồm 10 đơn mua hàng từ 6 nhà cung cấp.

Tổng giá trị đơn mua hàng trước thuế: 485.200.000 VND.

Kiểm tra sau import:

- Vendor.
- Order Date.
- Product.
- Quantity.
- Unit Price.
- Total.

Lưu ý: Import bản ghi Purchase Order không đồng nghĩa với việc hàng đã được nhập kho.

Để hoàn tất nghiệp vụ mua hàng:

1. Mở Purchase Order.
2. Kiểm tra sản phẩm và số lượng.
3. Confirm Order nếu đơn đang ở trạng thái RFQ.
4. Mở Receipt.
5. Kiểm tra số lượng thực nhận.
6. Nhấn Validate để xác nhận nhập kho.
7. Tạo Vendor Bill và Post hóa đơn nếu thực hiện đầy đủ quy trình kế toán.

### 13 – Sales Orders

**Mục đích:** Khởi tạo dữ liệu đơn bán hàng cho khách hàng.

Truy cập:

`Sales → Orders → Sales Orders`

Import file:

`13_sales_orders.csv`

Dữ liệu gồm 22 đơn bán hàng từ tháng 07 đến tháng 11/2026.

Tổng doanh thu trước thuế: 961.212.000 VND.

Kiểm tra sau import:

- Customer.
- Order Date.
- Pricelist.
- Product.
- Quantity.
- Unit Price.
- Discount.
- Total.

Lưu ý: Import Sales Order không đồng nghĩa với việc đơn hàng đã được xác nhận hoặc hàng đã được giao.

Để hoàn tất nghiệp vụ bán hàng:

1. Mở Sales Order.
2. Kiểm tra khách hàng và sản phẩm.
3. Kiểm tra Pricelist.
4. Nhấn Confirm nếu đơn đang ở trạng thái Quotation.
5. Xử lý Delivery Order nếu thực hiện nghiệp vụ xuất kho.
6. Tạo Customer Invoice nếu thực hiện nghiệp vụ kế toán.

---

## 9. Tạo Manufacturing Orders

Bộ CSV import từ 01 đến 13 không bao gồm file Manufacturing Orders riêng.

Các lệnh sản xuất được tạo trực tiếp trên Odoo sau khi đã cấu hình:

- Products.
- Work Centers.
- BoMs.
- Opening Stock.

Truy cập:

`Manufacturing → Operations → Manufacturing Orders`

Thực hiện:

1. Nhấn **New**.
2. Chọn Product.
3. Nhập Quantity.
4. Chọn BoM tương ứng.
5. Kiểm tra Components.
6. Nhấn Confirm.
7. Kiểm tra Work Orders.
8. Thực hiện các công đoạn sản xuất.
9. Hoàn tất lệnh bằng thao tác Produce All hoặc thao tác phù hợp với quy trình.

Dự án có 12 lệnh sản xuất mẫu từ tháng 07 đến tháng 11/2026.

Danh sách dữ liệu mẫu được quản lý riêng trong tài liệu/file `14_MO_Mau` nếu có trong repository.

---

## 10. Kiểm tra sau khi Import

Sau khi hoàn tất các bước, cần kiểm tra lại dữ liệu trong từng phân hệ.

| Phân hệ | Dữ liệu cần kiểm tra |
|---|---|
| Inventory | Product Categories, Products, Locations, Opening Stock |
| Manufacturing | Work Centers, BoMs, Manufacturing Orders |
| Contacts | Vendors, Customers |
| Sales | Pricelists, Sales Orders |
| Purchase | Purchase Orders |
| Employees | Employee Records |

### 10.1. Kiểm tra tồn kho

Truy cập:

`Inventory → Reporting`

Kiểm tra tồn kho theo từng Location.

Các vị trí cần xuất hiện:

- Kho Nhân xanh.
- Kho Bao bì.
- Xưởng sản xuất.
- Kho Thành phẩm.

### 10.2. Kiểm tra nghiệp vụ mua hàng

Đảm bảo các Purchase Orders có đúng nhà cung cấp, sản phẩm, số lượng và đơn giá.

Nếu thực hiện đầy đủ nghiệp vụ, kiểm tra Receipt đã Validate và Vendor Bill đã Post.

### 10.3. Kiểm tra nghiệp vụ bán hàng

Đảm bảo Sales Orders có đúng khách hàng, sản phẩm và Pricelist.

Kiểm tra riêng đơn hàng có quà tặng ly thủy tinh:

- Có dòng sản phẩm Ly thủy tinh 90S.
- Đơn giá bằng 0 VND.
- Số lượng quà tặng đúng dữ liệu mẫu.

### 10.4. Kiểm tra nghiệp vụ sản xuất

Đảm bảo các Manufacturing Orders có BoM, Components và Work Orders phù hợp.

Kiểm tra trạng thái hoàn tất sau khi thực hiện sản xuất.

---

## 11. Một số lệnh Docker thường dùng

Chạy hệ thống:

```bash
docker compose up -d
```

Dừng container:

```bash
docker compose stop
```

Khởi động lại:

```bash
docker compose start
```

Khởi động lại toàn bộ service:

```bash
docker compose restart
```

Xem trạng thái:

```bash
docker compose ps
```

Xem log:

```bash
docker compose logs -f
```

Dừng và xóa container/network được tạo bởi Compose:

```bash
docker compose down
```

**Cảnh báo:** Không sử dụng `docker compose down -v` nếu không có chủ đích xóa các named volumes. Thao tác này có thể xóa dữ liệu PostgreSQL và dữ liệu Odoo đã lưu trong volume.

---

## 12. Lưu ý khi làm việc nhóm

### 12.1. Dữ liệu CSV và dữ liệu Odoo là hai thứ khác nhau

Các file CSV trong `master_data` là bộ dữ liệu đầu vào dùng để tạo dữ liệu trong Odoo.

Việc đưa CSV lên GitHub không tự động đồng bộ các bản ghi đã import giữa các máy tính.

Mỗi thành viên cần:

1. Clone repository.
2. Khởi chạy Docker Compose.
3. Tạo database riêng.
4. Cài đặt ứng dụng.
5. Cấu hình hệ thống.
6. Import dữ liệu theo thứ tự 01–13.

### 12.2. Không commit dữ liệu database vào GitHub

Database PostgreSQL được lưu trong Docker volume `db-data`, không phải trong các file CSV.

Không nên commit trực tiếp thư mục dữ liệu database hoặc các Docker volumes vào repository.

### 12.3. Khi cập nhật Master Data

Nếu nhóm thay đổi dữ liệu:

1. Cập nhật file CSV tương ứng.
2. Kiểm tra dữ liệu và quan hệ tham chiếu.
3. Commit và push lên GitHub.
4. Thông báo cho các thành viên khác.
5. Các thành viên cập nhật repository và thực hiện import/cập nhật dữ liệu theo hướng dẫn.

Không import lại toàn bộ file một cách tùy tiện vì có thể tạo bản ghi trùng lặp.

---

## 13. Kết luận

Thông qua Docker Compose, dự án 90S COFFEE có thể triển khai môi trường Odoo 17 Community và PostgreSQL 15 thống nhất trên các máy tính thành viên.

Bộ Master Data CSV giúp chuẩn hóa dữ liệu ban đầu và hỗ trợ tái tạo môi trường ERP theo cùng một quy trình.

Các thành viên cần tuân thủ thứ tự import, kiểm tra dữ liệu sau mỗi bước và phân biệt giữa việc import bản ghi với việc xác nhận hoàn tất nghiệp vụ trong Odoo.
