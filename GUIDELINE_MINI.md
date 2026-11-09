# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp         | Định nghĩa của bạn                          | Ví dụ/điểm phân biệt                |
| ----------- | ------------------------------------------- | ----------------------------------- |
| `car` (0)   | Xe con: sedan, SUV, taxi, bán tải cabin nhỏ | Thân thấp, ngắn, có capo            |
| `truck` (1) | Xe chở hàng: tải thùng, tải nhẹ, đầu kéo    | Cabin tách rời thùng hàng           |
| `bus` (2)   | Xe buýt, xe khách, minibus                  | Thân dài, dãy cửa sổ khách liên tục |
| `van` (3)   | Van, minivan chở người/hàng                 | Thân hộp liền khối, ngắn hơn `bus`  |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: ôm sát phần thân xe nhìn thấy, không lấy bóng, mặt đường, xe bên cạnh; mỗi xe một box.
- Vật thể bị che khuất: box phần nhìn thấy; chỉ gán khi nhận ra lớp.
- Vật thể bị cắt bởi rìa ảnh: box chạm rìa ảnh, không đoán phần ngoài khung.
- Vật thể rất nhỏ/mơ hồ: gán nếu xác định được lớp; nếu không thì không đoán, ghi lại và hỏi giảng viên.
- Không gán: xe máy, xe đạp, người, xe trên biển quảng cáo/phản chiếu.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh         | Tình huống                                                           | Quyết định cuối                                                 | Lý do theo schema                                                   |
| ----------- | -------------------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------- |
| `drive_005` | Xe xám bị cắt rìa phải; xe trắng cabin + thùng; xe máy ở mép trái    | Xe xám `car` (box chạm rìa); xe trắng `truck`; xe máy không gán | Đủ nhận diện lớp; cabin tách thùng là xe tải; xe máy ngoài scope    |
| `drive_009` | Kẹt xe: xe buýt đỏ-trắng, xe tải thùng xanh, van trắng ở xa          | Buýt `bus`; tải xanh `truck`; van trắng `van`                   | Phân biệt theo độ dài thân và cabin tách/liền                       |
| `drive_045` | Dãy xe đỗ bên trái (bị cắt rìa, bị nhà chờ che) và nhiều xe nhỏ ở xa | Giữ xe trên lòng đường chính; bỏ dãy xe đỗ sau gold QC          | Gold không gán dãy này (có thể là vùng bỏ qua) nên đã bỏ khi rework |
