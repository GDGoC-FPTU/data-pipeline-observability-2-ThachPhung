# Experiment Report: Data Quality Impact on AI Agent

**Student ID:** 2A202601004
**Name:** Phùng Văn Thạch
**Date:** 10/06/2026

---

## 1. Ket qua thi nghiem

Chay `agent_simulation.py` voi 2 bo du lieu va ghi lai ket qua:

| Scenario | Agent Response | Accuracy (1-10) | Notes |
|----------|----------------|-----------------|-------|
| Clean Data (`processed_data.csv`) | Agent: Based on my data, the best choice is Laptop at $1200. | 9 | Ket qua dung: Laptop la san pham electronics co gia cao nhat (1200 USD), sau khi ETL da loai bo gia am va category rong. |
| Garbage Data (`garbage_data.csv`) | Agent: Based on my data, the best choice is Nuclear Reactor at $999999. | 2 | Ket qua sai hoan toan: Agent chon outlier gia 999999 USD thay vi san pham thuc te. |

**Cach chay thi nghiem:**

```bash
python3 solution.py          # Tao processed_data.csv (clean data)
python3 generate_garbage.py # Tao garbage_data.csv (poisoned data)
python3 agent_simulation.py   # Chay stress test voi ca 2 scenario
```

**Output thuc te khi chay `agent_simulation.py`:**

```
Testing with CLEAN data:
Agent: Based on my data, the best choice is Laptop at $1200.

Testing with GARBAGE data:
Agent: Based on my data, the best choice is Nuclear Reactor at $999999.
```

---

## 2. Phan tich & nhan xet

### Tai sao Agent tra loi sai khi dung Garbage Data?

Khi su dung `garbage_data.csv`, Agent tra loi sai vi logic cua no chi don gian tim san pham co gia cao nhat trong category "electronics" ma khong co bat ky buoc kiem tra chat luong du lieu nao. File garbage chua nhieu loi chat luong nghiem trong:

**Duplicate IDs:** Co hai record cung `id = 1` (Laptop va Banana). Dieu nay gay nham lan ve danh tinh ban ghi va lam Agent kho xac dinh du lieu nao la "nguon that".

**Wrong data types:** Gia cua "Broken Chair" la chuoi `"ten dollars"` thay vi so. Khi pandas doc CSV, cot price co the bi chuyen thanh kieu hon hop (object), lam phep so sanh `idxmax()` hoat dong khong on dinh hoac bo qua gia tri khong hop le mot cach khong du doan duoc.

**Extreme outliers:** "Nuclear Reactor" co gia 999999 USD — mot gia tri bat thuong ro rang nhung Agent van coi do la "best choice" vi no la gia cao nhat. Day la vi du dien hinh cua poisoned data: du lieu rac khong bi loc se keo leo ket qua ra ngoai thuc te.

**Null values:** Record "Ghost Item" co `id = None`, `category = None` va `price = 0`. Cac gia tri null lam mat tinh toan ven cua dataset va co the gay loi khi Agent loc theo category.

Nguoc lai, `processed_data.csv` da duoc ETL pipeline xu ly: loai bo gia <= 0, loai category rong, chuan hoa category thanh Title Case ("Electronics"), va them timestamp `processed_at`. Voi du lieu sach, Agent tim dung Laptop (1200 USD) — san pham electronics hop ly nhat trong tap du lieu.

Dieu nay cho thay **chat luong du lieu dau vao quyet dinh truc tiep do tin cay cua Agent**, bat ke prompt co ro rang den dau. Mot prompt tot khong the bu duoc cho du lieu bi nhiem rac, outlier, hay sai kieu.

---

## 3. Ket luan

**Quality Data > Quality Prompt?** Dong y.

Prompt chi huong Agent "tim san pham electronics gia cao nhat", nhung neu du lieu dau vao chua outlier hoac sai kieu, Agent van tra loi sai mot cach tu tin. ETL pipeline dong vai tro nhu "tam loc" truoc khi du lieu den Agent — giong nhu observability trong he thong thuc te: validate, transform, va log so record hop le/bi loai giup phat hien van de som thay vi de Agent "nghet" tren du lieu rac.

Trong bai lab nay, pipeline da loai 2/5 record (gia am va category rong), giu lai 3 record sach. Ket qua Agent voi clean data chinh xac, trong khi garbage data dan den cau tra loi vo nghia (Nuclear Reactor $999999). Do do, dau tu vao data quality va pipeline observability la nen tang bat buoc truoc khi trien khai bat ky ung dung AI nao dua tren du lieu noi bo.
