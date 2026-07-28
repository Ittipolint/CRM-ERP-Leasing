# Software Requirements Specification (SRS)
## ระบบ CRM-ERP-Leasing
## บริษัท Genive Green Co., Ltd.

---

## สารบัญ
1. [บทนำ](#1-บทนำ)
2. [ภาพรวมระบบ](#2-ภาพรวมระบบ)
3. [ข้อกำหนดด้านฟังก์ชันการทำงาน](#3-ข้อกำหนดด้านฟังก์ชันการทำงาน)
   - [3.1 ระบบฐานข้อมูล Company Profile](#31-ระบบฐานข้อมูล-company-profile)
   - [3.2 ระบบจัดทำ Quotation](#32-ระบบจัดทำ-quotation)
   - [3.3 ระบบฐานข้อมูล Prospect และ Customer](#33-ระบบฐานข้อมูล-prospect-และ-customer)
   - [3.4 ระบบฐานข้อมูลสินค้า (Product)](#34-ระบบฐานข้อมูลสินค้า-product)
4. [ข้อกำหนดด้านข้อมูล (Data Dictionary)](#4-ข้อกำหนดด้านข้อมูล-data-dictionary)
5. [ข้อกำหนดด้านความปลอดภัย](#5-ข้อกำหนดด้านความปลอดภัย)
6. [ข้อกำหนดด้านเทคนิค](#6-ข้อกำหนดด้านเทคนิค)

---

## 1. บทนำ

### 1.1 วัตถุประสงค์
เอกสารนี้อธิบายข้อกำหนดของระบบซอฟต์แวร์ CRM-ERP-Leasing สำหรับบริษัท Genive Green Co., Ltd. ซึ่งเป็นระบบบริหารจัดการลูกค้า ใบเสนอราคา สินค้า และข้อมูลบริษัทแบบครบวงจร

### 1.2 ขอบเขต
ระบบประกอบด้วย 4 โมดูลหลัก:
- ระบบฐานข้อมูลบริษัท (Company Profile)
- ระบบจัดทำใบเสนอราคา (Quotation)
- ระบบฐานข้อมูล Prospect และ Customer
- ระบบฐานข้อมูลสินค้า (Product)

### 1.3 ผู้ใช้งานระบบ
| บทบาท | คำอธิบาย |
|--------|----------|
| Admin | ผู้ดูแลระบบสูงสุด สามารถจัดการทุกส่วนได้ |
| Sales | พนักงานขาย จัดทำ Quotation และจัดการ Prospect/Customer |
| Manager | ผู้จัดการ อนุมัติ Quotation และดูรายงาน |
| Accountant | พนักงานบัญชี ดูข้อมูลการเงินจาก Quotation |

---

## 2. ภาพรวมระบบ

### 2.1 สถาปัตยกรรมระบบ
```
[Web Browser] → [Frontend (React/Next.js)] → [REST API] → [Backend (Node.js)] → [Database (PostgreSQL)]
```

### 2.2 ความสัมพันธ์ระหว่างโมดูล
```
Company Profile ──┐
                  ├──→ Quotation ──→ Customer
Product ──────────┘
Prospect ──→ Customer
```

---

## 3. ข้อกำหนดด้านฟังก์ชันการทำงาน

### 3.1 ระบบฐานข้อมูล Company Profile

#### 3.1.1 ข้อมูลบริษัท (Company Information)
| รหัส | รายการ | คำอธิบาย |
|------|--------|----------|
| CP-01 | จัดเก็บข้อมูลบริษัท | ชื่อบริษัท (ไทย/อังกฤษ), ที่อยู่ (ไทย/อังกฤษ), เลขประจำตัวผู้เสียภาษี, โทรศัพท์, อีเมล, เว็บไซต์, โลโก้ |
| CP-02 | แก้ไขข้อมูลบริษัท | ผู้ใช้สามารถแก้ไขข้อมูลบริษัทได้เฉพาะ Admin |
| CP-03 | แสดงข้อมูลบริษัท | แสดงข้อมูลบริษัทในรูปแบบที่พร้อมพิมพ์บน Quotation |

#### 3.1.2 Entity Relationship
```
Company (1) ──→ (N) CompanyBranch
Company (1) ──→ (N) CompanyContact
Company (1) ──→ (N) BankAccount
```

#### 3.1.3 ฟังก์ชันการทำงาน
- **CP-F01**: บันทึก/แก้ไขข้อมูลบริษัท
- **CP-F02**: จัดการสาขา (เพิ่ม/แก้ไข/ลบ)
- **CP-F03**: จัดการผู้ติดต่อของบริษัท
- **CP-F04**: จัดการบัญชีธนาคารของบริษัท (สำหรับแสดงใน Quotation)
- **CP-F05**: อัปโหลดรูปโลโก้บริษัท
- **CP-F06**: กำหนดค่าเริ่มต้นของระบบ เช่น เลขที่ Quotation อัตโนมัติ, ส่วนลดเริ่มต้น, เงื่อนไขการชำระเงิน

#### 3.1.4 ฟิลด์ข้อมูล
- `company_id` UUID PRIMARY KEY
- `company_name_th` VARCHAR(255) NOT NULL
- `company_name_en` VARCHAR(255)
- `tax_id` VARCHAR(13) NOT NULL UNIQUE
- `address_th` TEXT NOT NULL
- `address_en` TEXT
- `phone` VARCHAR(50)
- `email` VARCHAR(100)
- `website` VARCHAR(255)
- `logo_url` VARCHAR(500)
- `created_at` TIMESTAMP
- `updated_at` TIMESTAMP

---

### 3.2 ระบบจัดทำ Quotation

#### 3.2.1 ภาพรวม
ระบบจัดทำใบเสนอราคาที่รองรับทั้งรูปแบบ **ขายสด (Cash Sale)** และ **เช่า (Leasing)** โดยสามารถกำหนดระยะเวลาเช่า และคำนวณค่างวดรายเดือนได้อัตโนมัติ

#### 3.2.2 ประเภท Quotation
| ประเภท | คำอธิบาย |
|--------|----------|
| ขายสด (Cash) | ใบเสนอราคาสำหรับการขายสินค้าแบบชำระเงินครั้งเดียว |
| เช่า (Leasing) | ใบเสนอราคาสำหรับการเช่าสินค้า แบ่งชำระเป็นงวดรายเดือน |

#### 3.2.3 Entity Relationship
```
Quotation (1) ──→ (N) QuotationItem
Quotation (1) ──→ (1) Customer
Quotation (1) ──→ (1) Company
QuotationItem (N) ──→ (1) Product
```

#### 3.2.4 ฟังก์ชันการทำงาน
- **QT-F01**: สร้างใบเสนอราคาใหม่ (เลือกประเภท ขายสด/เช่า)
- **QT-F02**: เลือกลูกค้าจากฐานข้อมูล Customer
- **QT-F03**: เพิ่มสินค้าลงใน Quotation พร้อมระบุจำนวน ราคาต่อหน่วย ส่วนลด
- **QT-F04**: คำนวณราคารวมอัตโนมัติ
- **QT-F05**: กำหนดระยะเวลาเช่า (จำนวนเดือน) และคำนวณค่างวดต่อเดือน (สำหรับประเภทเช่า)
- **QT-F06**: สร้างเลขที่ Quotation อัตโนมัติตามรูปแบบที่กำหนด
- **QT-F07**: บันทึก Quotation เป็นฉบับร่าง (Draft)
- **QT-F08**: ส่ง Quotation ให้ผู้อนุมัติ (Submit for Approval)
- **QT-F09**: อนุมัติ/ปฏิเสธ Quotation (โดย Manager)
- **QT-F10**: พิมพ์ Quotation เป็น PDF
- **QT-F11**: ส่ง Quotation ทางอีเมลให้ลูกค้า
- **QT-F12**: ดูประวัติ Quotation ทั้งหมด พร้อมสถานะ
- **QT-F13**: ยกเลิก Quotation (Cancel)
- **QT-F14**: คัดลอก Quotation ฉบับเดิมเพื่อสร้างฉบับใหม่

#### 3.2.5 สถานะของ Quotation
```
Draft → Pending Approval → Approved → Sent → Converted to Order
                                              → Expired
                        → Rejected
Draft → Cancelled
```

#### 3.2.6 ฟิลด์ข้อมูล Quotation

**ตาราง quotations**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| quotation_id | UUID PK | รหัสอ้างอิง |
| quotation_no | VARCHAR(20) UNIQUE | เลขที่ใบเสนอราคา (เช่น QT-2024-00001) |
| quotation_type | ENUM('cash','leasing') | ประเภท ขายสด/เช่า |
| customer_id | UUID FK | รหัสลูกค้า |
| company_id | UUID FK | รหัสบริษัท (สำหรับกรณีมีหลายบริษัท) |
| status | ENUM('draft','pending','approved','rejected','sent','converted','cancelled','expired') | สถานะ |
| issue_date | DATE | วันที่ออกเอกสาร |
| valid_until | DATE | วันหมดอายุ |
| lease_term_months | INTEGER | ระยะเวลาเช่า (เดือน) เฉพาะประเภทเช่า |
| monthly_installment | DECIMAL(12,2) | ค่างวดต่อเดือน เฉพาะประเภทเช่า |
| subtotal | DECIMAL(12,2) | ราคาก่อนภาษี |
| discount_percent | DECIMAL(5,2) | ส่วนลดเป็นเปอร์เซ็นต์ |
| discount_amount | DECIMAL(12,2) | ส่วนลดเป็นจำนวนเงิน |
| vat_rate | DECIMAL(4,2) | อัตราภาษีมูลค่าเพิ่ม (เริ่มต้น 7%) |
| vat_amount | DECIMAL(12,2) | จำนวนภาษี |
| grand_total | DECIMAL(12,2) | ราคาสุทธิรวมภาษี |
| deposit_amount | DECIMAL(12,2) | เงินมัดจำ (สำหรับประเภทเช่า) |
| notes | TEXT | หมายเหตุ |
| terms_conditions | TEXT | เงื่อนไขการเสนอราคา |
| created_by | UUID FK | ผู้สร้าง |
| approved_by | UUID FK | ผู้อนุมัติ |
| approved_at | TIMESTAMP | วันที่อนุมัติ |
| created_at | TIMESTAMP | วันที่สร้าง |
| updated_at | TIMESTAMP | วันที่แก้ไขล่าสุด |

**ตาราง quotation_items**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| item_id | UUID PK | รหัสรายการ |
| quotation_id | UUID FK | รหัส Quotation |
| product_id | UUID FK | รหัสสินค้า |
| product_name | VARCHAR(255) | ชื่อสินค้า (บันทึก ณ เวลาที่เสนอราคา) |
| product_code | VARCHAR(50) | รหัสสินค้า |
| description | TEXT | รายละเอียดเพิ่มเติม |
| quantity | INTEGER | จำนวน |
| unit | VARCHAR(20) | หน่วย (เช่น ชุด, เครื่อง) |
| unit_price | DECIMAL(12,2) | ราคาต่อหน่วย |
| discount_percent | DECIMAL(5,2) | ส่วนลดรายการ (%) |
| discount_amount | DECIMAL(12,2) | ส่วนลดรายการ (บาท) |
| total_price | DECIMAL(12,2) | ราคารวม |
| sort_order | INTEGER | ลำดับรายการ |

#### 3.2.7 ตัวอย่างรูปแบบเลขที่ Quotation
- ปีงบประมาณ: `QT-{ปีพ.ศ.-1}-{เลขที่ 5 หลัก}` เช่น `QT-2567-00001`
- หรือแบบวันที่: `QT-{ปีพ.ศ.}{เดือน}-{เลขที่}` เช่น `QT-256701-001`

#### 3.2.8 การคำนวณ (Leasing)
```
ราคาสินค้ารวม = SUM(รายการสินค้าทั้งหมด)
ส่วนลด = ราคาสินค้ารวม × (discount_percent / 100) + discount_amount
มูลค่าก่อน VAT = ราคาสินค้ารวม - ส่วนลด
VAT = มูลค่าก่อน VAT × (vat_rate / 100)
ราคาสุทธิ = มูลค่าก่อน VAT + VAT
เงินมัดจำ (deposit) = ราคาสุทธิ × เปอร์เซ็นต์เงินมัดจำ (ถ้ามี)
ยอดคงเหลือ = ราคาสุทธิ - เงินมัดจำ
ค่างวดต่อเดือน = ยอดคงเหลือ / lease_term_months
```

---

### 3.3 ระบบฐานข้อมูล Prospect และ Customer

#### 3.3.1 ภาพรวม
ระบบจัดการข้อมูล Prospect (ลูกค้าที่กำลังจะซื้อ) และ Customer (ลูกค้าที่ซื้อแล้ว) โดย Prospect จะถูกแปลงเป็น Customer เมื่อมีการทำ Quotation สำเร็จ

#### 3.3.2 Entity Relationship
```
Prospect (N) ──→ (1) SalesPerson
Customer (1) ──→ (N) ContactPerson
Customer (1) ──→ (N) CustomerAddress
Customer (1) ──→ (N) Quotation
Prospect ──→ Customer (เมื่อเปลี่ยนสถานะ)
```

#### 3.3.3 ฟังก์ชันการทำงาน

**Prospect**
- **PR-F01**: เพิ่ม Prospect ใหม่ (ชื่อ, เบอร์โทร, อีเมล, ที่มา, หมายเหตุ)
- **PR-F02**: ค้นหา/กรอง Prospect (ตามชื่อ, เบอร์โทร, สถานะ, พนักงานขาย)
- **PR-F03**: แก้ไขข้อมูล Prospect
- **PR-F04**: แปลง Prospect เป็น Customer (เมื่อมีการซื้อ/ทำสัญญา)
- **PR-F05**: บันทึกประวัติการติดต่อ (Call Log / Activity Log)

**Customer**
- **CT-F01**: เพิ่มลูกค้าใหม่ (นิติบุคคล/บุคคลธรรมดา)
- **CT-F02**: ค้นหา/กรองลูกค้า
- **CT-F03**: แก้ไขข้อมูลลูกค้า
- **CT-F04**: จัดการผู้ติดต่อของลูกค้า (Contact Person)
- **CT-F05**: จัดการที่อยู่ของลูกค้า (หลายที่อยู่ได้)
- **CT-F06**: ดูประวัติ Quotation และใบเสร็จของลูกค้า
- **CT-F07**: ดูประวัติการติดต่อทั้งหมด
- **CT-F08**: ส่งออกรายงานลูกค้า (Excel/CSV/PDF)
- **CT-F09**: นำเข้าข้อมูลลูกค้าจากไฟล์ Excel/CSV

#### 3.3.4 สถานะของ Prospect
```
New → Contacted → Qualified → Converted to Customer
                 → Disqualified → Lost
```

#### 3.3.5 ฟิลด์ข้อมูล

**ตาราง prospects**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| prospect_id | UUID PK | รหัสอ้างอิง |
| first_name | VARCHAR(100) | ชื่อ |
| last_name | VARCHAR(100) | นามสกุล |
| company_name | VARCHAR(255) | ชื่อบริษัท (ถ้ามี) |
| phone | VARCHAR(20) | เบอร์โทรศัพท์ |
| email | VARCHAR(100) | อีเมล |
| source | ENUM('website','referral','phone','walk-in','event','social_media','other') | ที่มาของ Prospect |
| status | ENUM('new','contacted','qualified','disqualified','lost','converted') | สถานะ |
| assigned_to | UUID FK | พนักงานขายที่รับผิดชอบ |
| notes | TEXT | หมายเหตุ |
| converted_to_customer_id | UUID FK | รหัส Customer ที่แปลงแล้ว (nullable) |
| created_at | TIMESTAMP | วันที่สร้าง |
| updated_at | TIMESTAMP | วันที่แก้ไขล่าสุด |

**ตาราง customers**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| customer_id | UUID PK | รหัสอ้างอิง |
| customer_type | ENUM('individual','company') | ประเภทลูกค้า |
| first_name | VARCHAR(100) | ชื่อ (สำหรับบุคคลธรรมดา) |
| last_name | VARCHAR(100) | นามสกุล |
| company_name | VARCHAR(255) | ชื่อบริษัท (สำหรับนิติบุคคล) |
| tax_id | VARCHAR(13) | เลขประจำตัวผู้เสียภาษี |
| phone | VARCHAR(20) | เบอร์โทรศัพท์ |
| email | VARCHAR(100) | อีเมล |
| website | VARCHAR(255) | เว็บไซต์ |
| customer_code | VARCHAR(20) UNIQUE | รหัสลูกค้า |
| credit_term | INTEGER | ระยะเวลาเครดิต (วัน) |
| credit_limit | DECIMAL(12,2) | วงเงินเครดิต |
| payment_term | VARCHAR(100) | เงื่อนไขการชำระเงิน |
| notes | TEXT | หมายเหตุ |
| status | ENUM('active','inactive','blocked') | สถานะ |
| created_by | UUID FK | ผู้บันทึก |
| created_at | TIMESTAMP | วันที่สร้าง |
| updated_at | TIMESTAMP | วันที่แก้ไขล่าสุด |

**ตาราง customer_contacts**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| contact_id | UUID PK | |
| customer_id | UUID FK | |
| first_name | VARCHAR(100) | |
| last_name | VARCHAR(100) | |
| position | VARCHAR(100) | ตำแหน่ง |
| phone | VARCHAR(20) | |
| email | VARCHAR(100) | |
| is_primary | BOOLEAN | ผู้ติดต่อหลักหรือไม่ |

**ตาราง customer_addresses**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| address_id | UUID PK | |
| customer_id | UUID FK | |
| address_type | ENUM('billing','shipping','office','other') | |
| address | TEXT | |
| province | VARCHAR(100) | จังหวัด |
| district | VARCHAR(100) | อำเภอ |
| sub_district | VARCHAR(100) | ตำบล |
| zip_code | VARCHAR(10) | |
| is_default | BOOLEAN | |

**ตาราง activity_logs**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| log_id | UUID PK | |
| entity_type | ENUM('prospect','customer') | |
| entity_id | UUID | |
| activity_type | VARCHAR(50) | เช่น call, email, meeting, note |
| description | TEXT | |
| performed_by | UUID FK | |
| performed_at | TIMESTAMP | |

---

### 3.4 ระบบฐานข้อมูลสินค้า (Product)

#### 3.4.1 ภาพรวม
ระบบจัดการข้อมูลสินค้า/ผลิตภัณฑ์ของบริษัท รองรับการจัดหมวดหมู่ และการกำหนดราคาหลายแบบ (ขายสด/เช่า)

#### 3.4.2 Entity Relationship
```
ProductCategory (1) ──→ (N) Product
Product (1) ──→ (N) ProductPrice (ราคาแยกตามประเภท)
Product (1) ──→ (N) ProductImage
Product (1) ──→ (N) ProductSpecification
```

#### 3.4.3 ฟังก์ชันการทำงาน
- **PD-F01**: เพิ่มสินค้าใหม่ (รหัสสินค้า, ชื่อ, รายละเอียด, หมวดหมู่)
- **PD-F02**: ค้นหา/กรองสินค้า (ตามรหัส, ชื่อ, หมวดหมู่, สถานะ)
- **PD-F03**: แก้ไขข้อมูลสินค้า
- **PD-F04**: กำหนดราคาสินค้าแยกตามประเภท (ราคาขายสด, ราคาเช่าต่องวด)
- **PD-F05**: จัดการหมวดหมู่สินค้า (เพิ่ม/แก้ไข/ลบ)
- **PD-F06**: อัปโหลดรูปภาพสินค้า (หลายรูปได้)
- **PD-F07**: กำหนดสเปกสินค้า (เช่น ขนาดจอ, ความละเอียด, น้ำหนัก)
- **PD-F08**: กำหนดสถานะสินค้า (มีสินค้า/หยุดจำหน่าย/มาใหม่)
- **PD-F09**: ตั้งค่าราคาเช่าเริ่มต้น เช่น ราคาเช่าต่อเดือน
- **PD-F10**: นำเข้าสินค้าจากไฟล์ Excel/CSV
- **PD-F11**: ส่งออกรายการสินค้าเป็น Excel/CSV/PDF

#### 3.4.4 ฟิลด์ข้อมูล

**ตาราง product_categories**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| category_id | UUID PK | |
| category_name | VARCHAR(100) | ชื่อหมวดหมู่ |
| category_code | VARCHAR(20) | รหัสหมวดหมู่ |
| parent_id | UUID FK (self) | หมวดหมู่หลัก (สำหรับหมวดหมู่ย่อย) |
| description | TEXT | |
| sort_order | INTEGER | ลำดับการแสดงผล |
| is_active | BOOLEAN | |

**ตาราง products**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| product_id | UUID PK | |
| product_code | VARCHAR(50) UNIQUE | รหัสสินค้า |
| product_name | VARCHAR(255) | ชื่อสินค้า |
| product_name_en | VARCHAR(255) | ชื่อสินค้าภาษาอังกฤษ |
| description | TEXT | รายละเอียด |
| category_id | UUID FK | หมวดหมู่ |
| brand | VARCHAR(100) | ยี่ห้อ |
| model | VARCHAR(100) | รุ่น |
| unit | VARCHAR(20) | หน่วยนับ (ชุด, เครื่อง, ตัว) |
| warranty_term | VARCHAR(100) | ระยะเวลารับประกัน |
| stock_quantity | INTEGER | จำนวนคงเหลือ |
| min_stock_level | INTEGER | จำนวนขั้นต่ำที่ต้องมี |
| is_active | BOOLEAN | สถานะพร้อมขาย |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

**ตาราง product_prices**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| price_id | UUID PK | |
| product_id | UUID FK | |
| price_type | ENUM('cash','leasing_monthly') | ประเภทราคา |
| unit_price | DECIMAL(12,2) | ราคาต่อหน่วย |
| effective_date | DATE | วันที่เริ่มใช้ |
| end_date | DATE | วันที่สิ้นสุด (nullable) |
| is_current | BOOLEAN | เป็นราคาปัจจุบันหรือไม่ |

**ตาราง product_images**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| image_id | UUID PK | |
| product_id | UUID FK | |
| image_url | VARCHAR(500) | |
| sort_order | INTEGER | |
| is_primary | BOOLEAN | รูปหลัก |

**ตาราง product_specifications**
| ฟิลด์ | ประเภท | คำอธิบาย |
|------|--------|----------|
| spec_id | UUID PK | |
| product_id | UUID FK | |
| spec_name | VARCHAR(100) | ชื่อสเปก (เช่น "ขนาดจอ") |
| spec_value | VARCHAR(255) | ค่าสเปก (เช่น "86 นิ้ว") |
| sort_order | INTEGER | |

---

## 4. ข้อกำหนดด้านข้อมูล (Data Dictionary)

### 4.1 ความสัมพันธ์ระหว่างตาราง (ER Diagram - Textual)

```
┌─────────────────┐       ┌──────────────────────┐
│    companies    │      │    quotations         │
├─────────────────┤       ├──────────────────────┤
│ company_id (PK) │──┐   │ quotation_id (PK)     │
│ company_name_th │  └──→│ company_id (FK)       │
│ tax_id          │       │ customer_id (FK)     │
│ address_th      │       │ quotation_type       │
│ phone           │       │ status               │
│ email           │       │ grand_total          │
│ logo_url        │       │ lease_term_months    │
└─────────────────┘       │ monthly_installment  │
                          │ created_by (FK)      │
┌─────────────────┐       │ approved_by (FK)     │
│   customers     │       └────────┬─────────────┘
├─────────────────┤                │
│ customer_id(PK) │←───────────────┘
│ customer_type   │       ┌──────────────────────┐
│ company_name    │       │   quotation_items    │
│ tax_id          │       ├──────────────────────┤
│ phone           │       │ item_id (PK)         │
│ email           │       │ quotation_id (FK)    │
│ customer_code   │       │ product_id (FK)      │
│ status          │       │ product_name         │
└─────────────────┘       │ quantity             │
                          │ unit_price           │
┌─────────────────┐       │ total_price          │
│   prospects     │       └──────────────────────┘
├─────────────────┤
│ prospect_id(PK) │       ┌──────────────────────┐
│ first_name      │       │     products         │
│ last_name       │       ├──────────────────────┤
│ phone           │       │ product_id (PK)      │
│ email           │       │ product_code         │
│ status          │       │ product_name         │
│ converted_to    │       │ category_id (FK)     │
│   _customer_id  │       │ brand                │
└─────────────────┘       │ model                │
                          │ unit                 │
┌─────────────────┐       └────────┬─────────────┘
│ product_cat.    │                │
├─────────────────┤       ┌────────┴─────────────┐
│ category_id(PK) │       │  product_prices      │
│ category_name   │       ├──────────────────────┤
│ parent_id (FK)  │──────→│ price_id (PK)        │
└─────────────────┘       │ product_id (FK)      │
                          │ price_type           │
                          │ unit_price           │
                          │ effective_date       │
                          └──────────────────────┘
```

---

## 5. ข้อกำหนดด้านความปลอดภัย

| รหัส | รายการ | คำอธิบาย |
|------|--------|----------|
| SEC-01 | การยืนยันตัวตน (Authentication) | ใช้ JWT (JSON Web Token) สำหรับการล็อกอิน |
| SEC-02 | การกำหนดสิทธิ์ (Authorization) | กำหนดสิทธิ์ตาม Role (Admin, Sales, Manager, Accountant) |
| SEC-03 | การเข้ารหัสรหัสผ่าน | เข้ารหัสด้วย bcrypt |
| SEC-04 | การเข้ารหัสข้อมูลสำคัญ | ข้อมูลสำคัญเช่น Tax ID ควรเข้ารหัสในฐานข้อมูล |
| SEC-05 | Audit Log | บันทึกการแก้ไขข้อมูลสำคัญทั้งหมด (ใคร แก้อะไร เมื่อไหร่) |
| SEC-06 | การสำรองข้อมูล | สำรองข้อมูลอัตโนมัติทุกวัน |
| SEC-07 | HTTPS | ทุกการเชื่อมต่อต้องใช้ HTTPS |

### 5.1 สิทธิ์การเข้าถึงแบ่งตาม Role

| ฟังก์ชัน | Admin | Manager | Sales | Accountant |
|----------|-------|---------|-------|------------|
| จัดการ Company Profile | ✓ | - | - | - |
| ดู Company Profile | ✓ | ✓ | ✓ | ✓ |
| สร้าง/แก้ไข Quotation | ✓ | ✓ | ✓ | - |
| อนุมัติ Quotation | ✓ | ✓ | - | - |
| ยกเลิก Quotation | ✓ | ✓ | เฉพาะของตน | - |
| จัดการ Prospect | ✓ | ✓ | ✓ | - |
| จัดการ Customer | ✓ | ✓ | ✓ | ✓ |
| จัดการ Product | ✓ | ✓ | - | - |
| ดูรายงาน | ✓ | ✓ | เฉพาะของตน | เฉพาะการเงิน |
| จัดการผู้ใช้ | ✓ | - | - | - |

---

## 6. ข้อกำหนดด้านเทคนิค

| หัวข้อ | รายละเอียด |
|--------|------------|
| Frontend Framework | React.js / Next.js |
| Backend Framework | Node.js + Express หรือ NestJS |
| ฐานข้อมูล | PostgreSQL |
| ORM | Prisma หรือ TypeORM |
| Authentication | JWT + bcrypt |
| API Design | RESTful API |
| File Storage | Local หรือ Cloud (S3/Cloud Storage) |
| PDF Generation | Puppeteer หรือ PDFKit |
| การส่งอีเมล | Nodemailer หรือ SendGrid |
| Version Control | Git + GitHub |
| Containerization | Docker (optional) |

### 6.1 API Endpoints (概要)

#### Company Profile
```
GET    /api/v1/company              - ดึงข้อมูลบริษัท
PUT    /api/v1/company              - แก้ไขข้อมูลบริษัท
POST   /api/v1/company/branches     - เพิ่มสาขา
GET    /api/v1/company/branches     - ดึงรายการสาขา
POST   /api/v1/company/bank-accounts - เพิ่มบัญชีธนาคาร
```

#### Quotation
```
GET    /api/v1/quotations           - ดึงรายการ Quotation
GET    /api/v1/quotations/:id       - ดึง Quotation ตาม ID
POST   /api/v1/quotations           - สร้าง Quotation ใหม่
PUT    /api/v1/quotations/:id       - แก้ไข Quotation
PATCH  /api/v1/quotations/:id/status - เปลี่ยนสถานะ
DELETE /api/v1/quotations/:id       - ลบ Quotation (เฉพาะ Draft)
POST   /api/v1/quotations/:id/send-email - ส่งอีเมล
GET    /api/v1/quotations/:id/pdf   - ดาวน์โหลด PDF
```

#### Prospect
```
GET    /api/v1/prospects            - ดึงรายการ Prospect
POST   /api/v1/prospects            - เพิ่ม Prospect
PUT    /api/v1/prospects/:id        - แก้ไข Prospect
PATCH  /api/v1/prospects/:id/convert - แปลงเป็น Customer
POST   /api/v1/prospects/:id/activity - บันทึกกิจกรรม
```

#### Customer
```
GET    /api/v1/customers            - ดึงรายการ Customer
GET    /api/v1/customers/:id        - ดึง Customer ตาม ID
POST   /api/v1/customers            - เพิ่ม Customer
PUT    /api/v1/customers/:id        - แก้ไข Customer
GET    /api/v1/customers/:id/quotations - ดูประวัติ Quotation
POST   /api/v1/customers/import     - นำเข้าจาก Excel/CSV
GET    /api/v1/customers/export     - ส่งออกเป็น Excel/CSV
```

#### Product
```
GET    /api/v1/products             - ดึงรายการสินค้า
GET    /api/v1/products/:id         - ดึงสินค้าตาม ID
POST   /api/v1/products             - เพิ่มสินค้า
PUT    /api/v1/products/:id         - แก้ไขสินค้า
DELETE /api/v1/products/:id         - ลบสินค้า
GET    /api/v1/products/categories  - ดึงหมวดหมู่สินค้า
POST   /api/v1/products/categories  - เพิ่มหมวดหมู่สินค้า
```

---

## ภาคผนวก

### ก. ตัวอย่างฟอร์ม Quotation (ขายสด)
```
┌─────────────────────────────────────────────────┐
│              บริษัท Genive Green Co., Ltd.        │
│              123 หมู่ 4 ถนนสุขุมวิท              │
│              ตำบลบางนา อำเภอบางนา กรุงเทพฯ 10260 │
│              เลขผู้เสียภาษี: 0123456789012        │
│              โทร: 02-123-4567                    │
├─────────────────────────────────────────────────┤
│               ใบเสนอราคา / QUOTATION             │
│              เลขที่: QT-2567-00001               │
│              วันที่: 01/07/2567                   │
│              วันหมดอายุ: 31/07/2567              │
├─────────────────────────────────────────────────┤
│ ลูกค้า: องค์การบริหารส่วนตำบลศรีสำราญ           │
│ ที่อยู่: 999 หมู่ 3 ต.ศรีสำราญ อ.พรเจริญ        │
│         จ.บึงกาฬ                                 │
│ เลขผู้เสียภาษี: 0123456789012                    │
├─────────────────────────────────────────────────┤
│ ลำดับ │ รหัสสินค้า │ รายการ │ จำนวน │ ราคาต่อหน่วย │ จำนวนเงิน │
│   1   │ ID-86-01   │ จอ Interactive Display 86"│  1  │ 120,000.00 │ 120,000.00 │
│   2   │ ACC-01     │ ชุดอุปกรณ์ติดผนัง         │  1  │   5,000.00 │   5,000.00 │
├─────────────────────────────────────────────────┤
│ รวมราคาก่อนภาษี                    125,000.00  │
│ ภาษีมูลค่าเพิ่ม 7%                     8,750.00  │
│ ราคารวมทั้งสิ้น                    133,750.00  │
│                                            │
│ (หนึ่งแสนสามหมื่นสามพันเจ็ดร้อยห้าสิบบาทถ้วน)  │
├─────────────────────────────────────────────────┤
│ เงื่อนไข: ชำระเงินเต็มจำนวนก่อนส่งมอบ           │
│-------------------------------------------------│
│ ผู้มีอำนาจอนุมัติ _______________________________ │
│ วันที่อนุมัติ ___________                         │
└─────────────────────────────────────────────────┘
```

### ข. ตัวอย่างฟอร์ม Quotation (เช่า)
```
┌─────────────────────────────────────────────────┐
│              บริษัท Genive Green Co., Ltd.        │
│              123 หมู่ 4 ถนนสุขุมวิท              │
│              ตำบลบางนา อำเภอบางนา กรุงเทพฯ 10260 │
├─────────────────────────────────────────────────┤
│               ใบเสนอราคาเช่า / LEASING QUOTATION │
│              เลขที่: QT-2567-00002               │
│              ประเภท: เช่า 60 เดือน              │
├─────────────────────────────────────────────────┤
│ รายการสินค้า                                    │
│ 1. จอ Interactive Display 86" x 1 = 120,000.00 │
├─────────────────────────────────────────────────┤
│ ราคาสุทธิ:                        133,750.00    │
│ เงินมัดจำ 25%:                     33,437.50    │
│ ยอดคงเหลือ:                      100,312.50    │
│ ระยะเวลาเช่า: 60 เดือน                          │
│ ค่างวดต่อเดือน:                    1,671.88     │
│ (หนึ่งพันหกร้อยเจ็ดสิบเอ็ดบาทแปดสิบแปดสตางค์) │
├─────────────────────────────────────────────────┤
│ เงื่อนไข:                                       │
│ 1. ชำระเงินมัดจำก่อนส่งมอบ                      │
│ 2. ชำระค่างวดทุกวันที่ 5 ของเดือน                │
│ 3. สัญญาเช่ามีผลบังคับ 60 เดือน                  │
└─────────────────────────────────────────────────┘
```

---

> **หมายเหตุ:** เอกสารนี้เป็น Software Requirements Specification (SRS) เบื้องต้น สำหรับให้ทีมพัฒนาใช้เป็นแนวทางในการพัฒนาระบบ สามารถปรับแก้ไขเพิ่มเติมตามความเหมาะสม
>
> จัดทำโดย: ทีมวิศวกรรมซอฟต์แวร์
> วันที่: กรกฎาคม 2567
