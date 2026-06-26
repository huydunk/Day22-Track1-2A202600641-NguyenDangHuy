# Case 3 — Medical Call Summary & Routing Copilot — Workbook (Bài làm)

> Học viên: Nguyễn Đăng Huy — 2A202600641
> Case scaffold thấp: tự thiết kế workflow ASCII, UI ASCII, ≥2 checkpoint review, và **bắt buộc** có domain expert (màn hình + tiêu chí duyệt). Giá API dùng giá thật Anthropic (tháng 6/2026).

---

## 1. Unit of Work

**Quyết định:** Một cuộc gọi (transcript) của bệnh nhân/người nhà đi vào → AI (1) **tóm tắt** nội dung, (2) **phát hiện tín hiệu/red flag** y khoa (triệu chứng nặng, phản ứng thuốc), (3) **gợi ý route** về đúng nhóm: `hành chính/lịch hẹn`, `đơn thuốc`, `điều dưỡng sàng lọc`, `bác sĩ trực`, hoặc `quy trình khẩn cấp`. Output hiển thị cho **tổng đài viên** để họ quyết định chuyển đúng người/đúng quy trình.

**Lý do đây là đơn vị đủ nhỏ để eval nhưng vẫn chứa rủi ro lớn:**
- Ranh giới rõ: *một cuộc gọi vào → một bộ {summary, red_flag, route} ra*, chấm được từng case độc lập.
- Vẫn chạm rủi ro cao nhất: nếu bỏ sót red flag hoặc route sai, bệnh nhân có thể **bị xử lý chậm/sai → nguy hiểm sức khỏe**, hoặc hệ thống **lộ nhầm hồ sơ y tế**.
- **Không** chọn "toàn bộ tổng đài y tế" / "trợ lý y khoa" — quá rộng, gộp nhiều bước và con người, không rõ đang đo gì.
- Ranh giới quan trọng: AI **chỉ tóm tắt + gợi ý route**, **không tự chẩn đoán**, **không tự trả lời thay bác sĩ**.

---

## 2. Quality Question

**Câu hỏi chất lượng:** Với mỗi cuộc gọi, AI có (a) **phân biệt đúng** cuộc gọi hành chính/đơn thuốc với cuộc gọi cần nhân sự y khoa can thiệp, (b) **escalate đúng** sang `điều dưỡng`/`bác sĩ`/`quy trình khẩn cấp` khi xuất hiện red flag, và (c) **không vượt ranh giới an toàn** (không chẩn đoán, không bung hồ sơ khi chưa xác định đúng bệnh nhân) hay không?

**Vì sao fail ở đây là nghiêm trọng:**
- Transcript có "khó thở/đau ngực/tím tái" mà bị route về `CSKH đơn thuốc`, `red_flag=false` → **bỏ sót cấp cứu**, có thể gây hại tính mạng (đúng lỗi mock outcome của đề).
- Tóm tắt **làm nhẹ** mức nghiêm trọng → tổng đài viên chủ quan, xử lý chậm.
- Lookup sai bệnh nhân rồi **bung hồ sơ y tế** → vi phạm quyền riêng tư, rủi ro pháp lý.

Hành vi **bắt buộc**: escalate ngay khi có red flag; phân tách rõ *điều bệnh nhân nói* / *điều tra cứu được* / *điều AI suy luận*. Hành vi **bị cấm**: tự chẩn đoán, đưa chỉ định điều trị, lộ hồ sơ khi chưa chắc đúng người.
