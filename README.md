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
├── assets/
│   ├── css/         # style.css (Professional ERP UI)
│   ├── js/          # script.js
│   └── img/         # โลโก้บริษัท
└── modules/
    ├── company/     # จัดการข้อมูลบริษัท (Company Profile, สาขา, บัญชีธนาคาร)
    ├── quotation/   # ใบเสนอราคา (ขายสด/เช่า) พร้อม Print
    ├── prospect/    # โอกาสขาย (Leads)
    ├── customer/    # ข้อมูลลูกค้า
    ├── product/     # สินค้าและหมวดหมู่สินค้า
    └── report/      # รายงาน
```

## UI/UX การออกแบบ

ระบบออกแบบด้วยโทน Professional ERP สีเขียว (Genive Green):
- **Sidebar:** สีกรมท่า (#1a2035) พร้อมโลโก้บริษัท
- **Primary Color:** #059669 (เขียว) พร้อม Hover Effect
- **Cards:** มุมโค้งมน 14px, เงา Hover Effect
- **Tables:** หัวตารางพื้นสีอ่อน, แถว Hover
- **Dashboard:** การ์ดขนาดเท่ากันด้วยระบบ Grid Auto-layout (row-cols)
- **Responsive:** รองรับทุกขนาดหน้าจอ (Mobile-first)
- **Login:** Gradient Background, Floating Labels

## Changelog

| วันที่ | การเปลี่ยนแปลง |
|-------|---------------|
| 2026-07-29 | **UI Overhaul:** Professional ERP Theme, โลโก้ Genive Green, Dashboard การ์ดเท่ากัน |
| 2026-07-29 | **Fix:** SQL Column Mismatches, Relative Paths, CRUD Operations ทั้งหมด |
| 2026-07-29 | **Feature:** Product Categories, Company Branches, Bank Accounts |
| 2026-07-29 | **Fix:** FK ON DELETE Rules (RESTRICT for products.category_id & quotation_items.product_id) |
| 2026-07-29 | **Fix:** Remove PHP dual-writes in company/edit, quotation/create, quotation/edit |
| 2026-07-29 | **Feature:** Active/Inactive Toggle สำหรับ Master Data (สินค้า, หมวดหมู่สินค้า, ลูกค้า) |
| 2026-07-29 | **Feature:** เพิ่ม/แก้ไขหมวดหมู่สินค้ามีฟิลด์สถานะ (is_active) ใน Modal |
| 2026-07-29 | **Fix:** ซ่อน Inactive Reference Data จาก Dropdown (product create/edit กรองเฉพาะ is_active=1) |
| 2026-07-29 | **Fix:** แก้ไข INSERT quotations column mismatch (มี `?` เกิน 1 ตัว) |
| 2026-07-29 | **Feature:** Product Dropdown ใน Quotation create/edit พร้อม Auto-fill ราคา |
| 2026-07-29 | **Fix:** calculateRow ใช้ [name^=item_qty] แทน .auto-calc เพื่อให้คำนวณถูกต้องทุกแถว |
| 2026-07-29 | **Feature:** Print quotation แสดงคอลัมน์รายละเอียด |
| 2026-07-29 | **Fix:** addItemRow ใช้ insertRow + template innerHTML หุ้ม <select> เพื่อให้ dropdown ทุกแถว |
| 2026-07-29 | **Test:** 23 constraint/CRUD tests all pass |
| 2026-07-30 | **Feature:** ย้าย product dropdown ไปไว้หัวตาราง quotation พร้อมปุ่ม Add เลือก code ลงตาราง |
| 2026-07-30 | **Feature:** เพิ่ม field รายละเอียด (description) ใน create/edit/view/print quotation |
| 2026-07-30 | **Feature:** คลิกแถวใน view quotation → ไปหน้าแก้ไข |
| 2026-07-30 | **Fix:** ลบ required จาก readOnly fields, เพิ่ม novalidate ป้องกัน browser validation block |
