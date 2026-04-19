---
name: xu-ly-sach-moi
description: Phân tích sách Thái Ất mới và tổng hợp kiến thức vào Obsidian knowledge-base. Dùng khi có folder sách mới được thêm vào repo.
argument-hint: <tên-thư-mục-sách-mới>
allowed-tools: Bash(find *) Bash(ls *) Bash(wc *) Bash(git *) Read Glob Grep Write Edit Agent
---

# Skill: Xử Lý Sách Mới Thái Ất

Khi được gọi với `/xu-ly-sach-moi <tên-thư-mục>`, thực hiện quy trình sau để phân tích sách và đồng bộ vào knowledge-base.

## BƯỚC 0 — Khám phá sách mới

Tìm tất cả các file MD trong thư mục sách được chỉ định:

```bash
find /home/user/thai-at-the-scholar/"$ARGUMENTS" -name "*.md" | sort
```

Nếu `$ARGUMENTS` trống, hỏi người dùng tên thư mục sách.

**Lấy thông tin tổng quan:**
- Tổng số file MD
- Tên file đầu tiên (thường là trang bìa/mục lục)
- Dung lượng tổng (để ước tính khối lượng)

**Quy tắc xử lý theo quy mô:**
- **≤ 20 files** → đọc trực tiếp tất cả trong session hiện tại
- **21–60 files** → chia 3 batch, dùng 3 Explore subagents song song
- **61–120 files** → chia 4–6 batch, dùng tối đa 3 Explore subagents song song, chạy nhiều lượt
- **> 120 files** → chia thành các nhóm chapter, xử lý tuần tự từng nhóm

## BƯỚC 1 — Đọc cấu trúc sách (Survey Pass)

Đọc **5 file đầu tiên** để nắm mục lục và cấu trúc chương:
- Trang bìa, lời tựa, mục lục
- Xác định: số chương, số phần, các chủ đề chính
- Ghi nhận tên tác giả, năm xuất bản, nguồn gốc

Dựa vào mục lục, lập danh sách **batch plan** — mỗi batch là 10–15 files liên tiếp, đặt tên theo chương/chủ đề.

Ví dụ:
```
Batch 1 (P6–P15):  Chương 1 — Cơ chế và nền tảng
Batch 2 (P16–P25): Chương 2 — Phép tính cơ bản
Batch 3 (P26–P40): Chương 3 — Ứng dụng và ví dụ
...
```

## BƯỚC 2 — Deep Read (Song song theo batch)

Với mỗi batch, spawn một **Explore subagent** với prompt sau:

```
Đọc các file sau trong repo thai-at-the-scholar:
[danh sách đường dẫn file của batch]

Nhiệm vụ: Trích xuất và tổ chức thông tin theo các danh mục:

1. **KHÁI NIỆM MỚI** — Định nghĩa, thuật ngữ chưa thấy trong knowledge-base hiện tại
2. **THỰC THỂ** — Thần sát, cung vị, sao, tướng mới (tên tiếng Việt + Hán tự)
3. **PHÉP TÍNH / QUY TẮC** — Công thức, bước thực hiện mới
4. **CÁCH CỤC** — Mẫu hình mới chưa được đề cập
5. **VÍ DỤ MINH HỌA** — Ví dụ có số liệu cụ thể, case study
6. **BỔ SUNG CHO NOTES HIỆN CÓ** — Thông tin mới về khái niệm đã tồn tại trong knowledge-base

Knowledge-base hiện có tại: /home/user/thai-at-the-scholar/knowledge-base/
Các notes hiện có: [liệt kê tên files bằng Glob]

Với mỗi mục, ghi rõ: tên khái niệm, nội dung tóm tắt, file nguồn (P số mấy), và gợi ý note nào trong knowledge-base nên tạo hoặc cập nhật.

Trả về một báo cáo có cấu trúc rõ ràng.
```

**Chạy tối đa 3 subagents song song trong một lần.** Nếu có nhiều hơn 3 batches, chờ lượt trước hoàn thành rồi chạy lượt tiếp.

## BƯỚC 3 — Tổng hợp kết quả

Sau khi tất cả batch hoàn thành, tổng hợp:

**3a. Danh sách NOTES CẦN TẠO MỚI:**
```
Tên note | Thư mục | Nội dung cốt lõi | Batch nguồn
```

**3b. Danh sách NOTES CẦN CẬP NHẬT:**
```
File hiện có | Thông tin bổ sung | Batch nguồn
```

**3c. Danh sách MOC CẦN CẬP NHẬT:**
- Xác định MOC nào trong `00-Tong-Quan/` cần thêm link mới

## BƯỚC 4 — Tạo/cập nhật notes

### Tạo Source Book Note

Tạo file `knowledge-base/07-Nguon-Sach/<TenSach>.md` theo template:

```yaml
---
title: "<Tên đầy đủ>"
aliases: ["<Hán tự>", "<Tên ngắn>"]
tags: [thai-at, nguon-sach]
level: reference
type: source
tac-gia: ""
nam-dich: ""
so-trang: 0
nguon-goc: ""
so-files-md: <số file>
created: <ngày hôm nay>
---
```

### Tạo Notes Khái Niệm Mới

Với mỗi khái niệm/thực thể/phương pháp/cách cục mới:
- Dùng template phù hợp trong `_Templates/`
- Đặt vào đúng thư mục (01–06)
- Điền đầy đủ frontmatter (level, type, chinese, sources, related)
- Viết nội dung bằng tiếng Việt, súc tích nhưng đủ ý
- Thêm links `[[WikiLink]]` đến các notes liên quan

**Quy tắc đặt tên file:** kebab-case tiếng Việt không dấu, ví dụ: `Bach-Tuyet-Quang-Minh.md`

### Bổ sung Notes Hiện Có

Với mỗi note cần cập nhật, dùng `Edit` để thêm thông tin mới vào phần phù hợp. Không xóa nội dung cũ. Thêm source mới vào frontmatter `sources`.

### Cập nhật MOC Files

Với mỗi note mới:
- Thêm link vào MOC tương ứng (`MOC-Cac-Than.md`, `MOC-Phep-Tinh.md`...)
- Nếu note mới thuộc một nhóm chưa có trong MOC → thêm nhóm mới

## BƯỚC 5 — Kiểm tra chất lượng

Trước khi commit, kiểm tra:
- [ ] Tất cả `[[WikiLink]]` trong notes mới có file tương ứng tồn tại
- [ ] Frontmatter đầy đủ: `title`, `tags`, `level`, `type`, `sources`, `created`
- [ ] Source book note đã có danh sách khái niệm được tạo từ sách này
- [ ] Ít nhất 1 MOC đã được cập nhật với link đến notes mới

Kiểm tra links bị hỏng:
```bash
grep -r '\[\[' /home/user/thai-at-the-scholar/knowledge-base/ --include="*.md" | grep -oP '\[\[([^\]]+)\]\]' | sort | uniq
```

## BƯỚC 6 — Commit

```bash
git add knowledge-base/
git commit -m "feat(sách): thêm kiến thức từ <Tên Sách>

- X notes mới tạo
- Y notes cập nhật  
- Z MOC files cập nhật

https://claude.ai/code/session_015ntJ85VdcrZ9igs8gKHaEi"
```

```bash
git push -u origin claude/thai-at-knowledge-library-pcomi
```

## BƯỚC 7 — Báo cáo kết quả

Sau khi hoàn thành, tóm tắt cho người dùng:

```
✅ Xử lý xong: <Tên Sách>

📚 Đã tạo: X notes mới
  - N notes trong 01-Nen-Tang
  - N notes trong 02-Cac-Than
  - ...

🔄 Đã cập nhật: Y notes hiện có
📋 Đã cập nhật: Z MOC files

🆕 Khái niệm nổi bật mới:
  - Tên khái niệm 1: mô tả ngắn
  - Tên khái niệm 2: mô tả ngắn
  ...
```

---

## Lưu ý xử lý sách lớn (> 300 trang / > 60 files)

- **Không đọc tất cả files cùng lúc** — context sẽ tràn
- Luôn bắt đầu bằng Survey Pass (5 files đầu) để lập batch plan
- Ghi kết quả tổng hợp của từng batch vào scratch notes tạm thời trong response trước khi chuyển sang batch tiếp theo
- Ưu tiên **khái niệm mới hoàn toàn** hơn là bổ sung chi tiết nhỏ vào notes cũ
- Nếu một chapter quá dài và phức tạp → tạo 1 "overview note" cho chapter đó, chi tiết để lại cho lần sau

## Cấu trúc knowledge-base hiện tại

```
knowledge-base/
├── 00-Tong-Quan/   ← MOC và lộ trình học
├── 01-Nen-Tang/    ← Khái niệm cơ bản (beginner)
├── 02-Cac-Than/    ← Thần sát, tinh thần (intermediate)
├── 03-Cac-Cung/    ← Cửu Cung chi tiết (intermediate)
├── 04-Phep-Tinh/   ← Phép tính, công thức (intermediate)
├── 05-Cach-Cuc/    ← Cách cục, mẫu hình (advanced)
├── 06-Ung-Dung/    ← Ứng dụng thực tế (advanced)
├── 07-Nguon-Sach/  ← Source book notes
└── _Templates/     ← Templates để tạo notes mới
```
