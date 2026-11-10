# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con, sedan, hatchback, SUV, xe taxi, xe bán tải (pickup truck). | Các phương tiện cá nhân 4-7 chỗ, xe bán tải có cabin nhỏ chở người (như Ford Ranger, Hilux). Không gán xe buýt, xe tải chở hàng lớn, xe mô tô/xe máy. |
| `truck` | Xe tải chở hàng cỡ trung và lớn, xe tải thùng, xe ben, xe bồn, xe container/đầu kéo. | Phương tiện chuyên dụng chở hàng hóa nặng, có cabin lái tách biệt với khoang chở hàng/thùng xe phía sau. Không gán xe bán tải nhỏ (thuộc `car`). |
| `bus` | Xe buýt công cộng, xe khách liên tỉnh, xe du lịch cỡ lớn, minibus chở khách công cộng. | Xe chở nhiều người (thường từ 16 chỗ trở lên), thân xe dài, nhiều cửa sổ hành khách lớn. Không gán xe van chở hàng hoặc minivan gia đình (thuộc `van`). |
| `van` | Xe van, minivan, MPV chở hàng hoặc chở người cỡ nhỏ/trung. | Thân xe dạng hộp liền khối, đuôi phẳng, thường có cửa trượt bên hông (ví dụ: Ford Transit, Toyota Hiace, xe van giao hàng nhỏ). Phân biệt với `car` thông thường và `bus` cỡ lớn. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding box phải ôm khít các mép rìa ngoài cùng của thân xe (tight boundary), bao gồm cả bánh xe tiếp đất và gương chiếu hậu. Tuyệt đối không để thừa diện tích mặt đường, khoảng không bầu trời hoặc lấn sang xe bên cạnh; không gộp hai xe vào một box.
- Vật thể bị che khuất: Vẽ bounding box chỉ bao quanh phần nhìn thấy được (visible part) của vật thể, miễn là phần lộ diện đủ đặc điểm để nhận diện đúng lớp. Không vẽ bao trùm hoặc đoán phần thân xe bị vật cản (cột đèn, cây cối, xe khác) che lấp.
- Vật thể bị cắt bởi rìa ảnh: Cạnh của bounding box chạm đúng mép biên của khung hình (`0.0` hoặc `1.0`). Tuyệt đối không mở rộng tọa độ ra ngoài khung hình và không đoán kích thước phần xe nằm ngoài ảnh.
- Vật thể rất nhỏ/mơ hồ: Những xe ở quá xa phía hậu cảnh (kích thước < 10-15 pixels) hoặc bị nhòe/mờ nghiêm trọng do mưa/cảm biến mà không thể phân biệt chắc chắn là loại xe gì thì KHÔNG tự đoán và không gán nhãn để tránh sinh False Positive (dương tính giả).

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe ô tô di chuyển ở làn giữa bị cột đèn đường thẳng đứng che khuất một phần thân xe phía trước. | Gán nhãn `car` (class 0), vẽ box ôm sát phần thân xe nhìn thấy được từ mép cột đèn trở về sau. | Vật thể nhận diện rõ là xe SUV/sedan (`car`). Quy tắc nêu rõ: xe bị che khuất thì vẽ box ôm sát phần nhìn thấy, không mở rộng đoán phần thân xe bị cột đèn che. |
| `drive_005.jpg` | Chiếc xe màu xám ở góc dưới cùng bên phải màn hình bị cắt ngang bởi mép ảnh dưới và phải (chỉ thấy phần đuôi xe). | Gán nhãn `car` (class 0), vẽ box chạm sát cạnh dưới và cạnh phải của ảnh. | Nhận diện rõ đặc điểm đèn hậu và cốp sau của xe con (`car`). Áp dụng quy tắc cắt khung: box chạm đúng rìa ảnh, không tự đoán kích thước phần đầu xe ngoài khung hình. |
| `drive_001.jpg` | Hàng loạt xe đang đỗ dọc bên lề đường và bãi đỗ phía bên trái (xe màu trắng, xe xám). | Vẫn gán nhãn `car` (class 0) riêng biệt cho từng xe đỗ nhìn rõ hình dạng. | Schema phân loại mô tả bản chất loại vật thể (object type), không phân biệt trạng thái động học (đang chạy hay đang đỗ). Các xe đỗ vẫn là đối tượng chướng ngại vật cần phát hiện trong hệ thống ADAS. |
