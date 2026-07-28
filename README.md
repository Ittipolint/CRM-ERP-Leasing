# CRM-ERP-Leasing System

ระบบบริหารจัดการลูกค้า ใบเสนอราคา สินค้า และข้อมูลบริษัท สำหรับ **บริษัท Genive Green Co., Ltd.**

## โมดูลหลัก

1. **Company Profile** - ระบบฐานข้อมูลบริษัท
2. **Quotation** - ระบบจัดทำใบเสนอราคา (ขายสด/เช่า)
3. **Prospect & Customer** - ระบบฐานข้อมูลลูกค้า
4. **Product** - ระบบฐานข้อมูลสินค้า

## เอกสาร

- [Software Requirements Specification (SRS)](docs/SRS_CRM-ERP-Leasing.md)
- [เอกสารตัวอย่าง (PDF)](Documents/)

## Tech Stack

- **OS:** Ubuntu (รันบน VirtualBox)
- **Web Server:** Apache
- **Backend:** PHP 8.x (Core PHP)
- **Database:** MySQL (MariaDB)
- **Frontend:** HTML5 + CSS3 (Responsive Design with Bootstrap 5)

## การติดตั้ง (ระบบพัฒนาแล้ว)

ระบบ CRM-ERP-Leasing ได้ถูกพัฒนาตาม SRS และติดตั้งบน VM VirtualBox แล้ว

### สภาพแวดล้อมการทำงานจริง
- **VM:** Ubuntu 26.04 บน VirtualBox
- **IP:** 192.168.1.54
- **Web:** http://192.168.1.54/crm-erp-leasing/
- **User:** admin / admin123

### Tech Stack ที่ใช้
- **OS:** Ubuntu (VirtualBox)
- **Web Server:** Apache
- **Backend:** PHP 8.5 (Core PHP)
- **Database:** MySQL 8.4
- **Frontend:** HTML5 + CSS3 (Bootstrap 5, Responsive)

### โครงสร้างระบบ
```
/var/www/html/crm-erp-leasing/
├── config/          # การตั้งค่า DB และระบบ
├── includes/        # Header, Footer, Auth, Functions
├── auth/            # Login, Logout, เปลี่ยนรหัสผ่าน
├── assets/          # CSS, JS, รูปภาพ
└── modules/
    ├── company/     # จัดการข้อมูลบริษัท
    ├── quotation/   # ใบเสนอราคา (ขายสด/เช่า)
    ├── prospect/    # โอกาสขาย
    ├── customer/    # ข้อมูลลูกค้า
    ├── product/     # สินค้าและหมวดหมู่
    └── report/      # รายงาน
```
