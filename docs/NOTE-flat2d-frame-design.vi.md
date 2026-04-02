# Ghi chú cho designer khi thiết kế frame Flat2D

## Mục tiêu

Tài liệu này giúp designer bàn giao asset khung sao cho artwork luôn hiển thị đúng bên trong khung khi render bằng Flat2D framing.

## Kết luận ngắn

- Renderer 9-slice hiện tại **không vẽ vùng `center`** của asset khung.
- Vì vậy, artwork sẽ hiển thị qua phần lòng khung dù file PNG gốc có center opaque.
- Tuy nhiên, để handoff sạch và an toàn cho các use case tương lai, **vẫn nên thiết kế center là transparent** và export PNG có alpha.

## Designer chỉ cần bàn giao gì

Designer **không cần**:

- cắt sẵn asset thành 9 file riêng
- chuẩn bị nhiều phiên bản cho từng mức zoom
- làm riêng asset cho pinch gesture

Designer **chỉ cần** bàn giao tối thiểu:

- 1 file PNG master của toàn bộ frame
- 1 thumbnail nếu muốn dùng ảnh preview khác ảnh master
- độ dày phần viền của khung ở 4 cạnh: trái, trên, phải, dưới
- đánh dấu phần nào của khung có thể kéo dài ra mà vẫn đẹp, phần nào phải giữ nguyên
- độ dày mặt khung mong muốn ngoài đời, ví dụ 2 cm hoặc 3 cm

Designer không cần biết thuật ngữ kỹ thuật của app. Chỉ cần hiểu đơn giản:

- dev cần biết phần viền khung dày bao nhiêu ở mỗi cạnh
- dev cần biết chỗ nào là chi tiết cố định như góc, hoa văn, mối nối
- dev cần biết chỗ nào là phần cạnh có thể kéo dài khi tranh to hơn hoặc nhỏ hơn

Pinch zoom là hành vi runtime của app. Khi user dùng 2 ngón tay để phóng to hoặc thu nhỏ, app sẽ scale **cả cụm framed artwork** như một khối thống nhất. Designer không cần xuất thêm asset riêng cho trường hợp này.

## Ví dụ designer cần gửi cho dev với 1 thiết kế frame

Ví dụ designer làm một mẫu khung gỗ tên là `classic-oak-frame.png`.

Designer chỉ cần gửi cho dev:

- file `classic-oak-frame.png`
- nếu muốn, thêm 1 ảnh preview nhỏ để hiện trong danh sách chọn khung
- một note ngắn ngay trên Figma hoặc trong message handoff như sau:

```text
Tên mẫu: Classic Oak

Độ dày viền:
- trái: 18 px
- trên: 18 px
- phải: 18 px
- dưới: 18 px

Phần phải giữ nguyên:
- 4 góc
- phần hoa văn gần góc

Phần có thể kéo dài:
- đoạn cạnh thẳng ở giữa mỗi cạnh

Lòng khung:
- để rỗng / trong suốt để thấy tranh bên trong

Cảm giác độ dày ngoài đời:
- khoảng 3 cm
```

Chỉ cần như vậy là dev đã có đủ thông tin để nối vào app.

## Designer cần lưu ý

### 1. Hãy thiết kế theo nguyên tắc: góc giữ nguyên, cạnh giữa có thể kéo dài

Asset khung cần tách được thành:

- 4 góc
- 4 cạnh
- 1 vùng giữa

4 góc phải giữ chi tiết tốt khi không stretch. 4 cạnh phải chịu được stretch theo 1 chiều mà không bị méo texture hoặc pattern.

### 2. Vùng giữa nên là phần rỗng của khung

- Phần giữa nên được coi là "lỗ mở" để nhìn thấy artwork.
- Khuyến nghị export PNG với **alpha trong suốt ở center**.
- Không nên đặt họa tiết quan trọng, bóng kính, glare, dust, bevel chờm vào center nếu mong muốn artwork nhìn rõ.

Lưu ý: ở runtime hiện tại, vùng `center` không được vẽ. Nếu designer đặt hiệu ứng trong center thì hiệu ứng đó sẽ không xuất hiện.

### 3. Không dùng center để tạo hiệu ứng đè lên artwork

Nếu cần các hiệu ứng như:

- phản chiếu kính
- glare
- bụi mặt kính
- inner shadow chờm lên tranh

thì các hiệu ứng này không nên nằm trong center của frame asset. Chúng cần được:

- đưa vào border của frame nếu chỉ nằm trên mép khung, hoặc
- tách thành layer khác nếu muốn phủ lên artwork

### 4. Cần nói rõ phần viền nào giữ nguyên, phần nào có thể kéo dài

Khi handoff, designer nên cung cấp:

- file PNG chính của khung
- preview thumbnail nếu cần khác file chính
- số đo độ dày viền của khung ở 4 cạnh: trái, trên, phải, dưới
- ghi chú rõ phần nào được phép kéo dài khi đổi kích thước, phần nào phải giữ nguyên chi tiết
- độ dày mặt khung mong muốn ngoài đời, ví dụ 2 cm hoặc 3 cm

Ví dụ cách ghi chú dễ hiểu cho designer:

- viền trái: 18 px
- viền trên: 18 px
- viền phải: 18 px
- viền dưới: 18 px
- 4 góc và phần hoa văn gần góc phải giữ nguyên
- phần cạnh thẳng ở giữa có thể kéo dài

Hiện tại dev map các thông số này ở `riz-app-v2/screens/create-flat2d-framing/constants/frame-presets.ts`.

### 5. Tránh để chi tiết quan trọng sát đường slice

Các chi tiết như:

- vân gỗ đặc biệt
- mối nối hoa văn
- gờ nổi mạnh
- viền kim loại có highlight sắc

không nên đặt sát ranh giới slice nếu chúng không chịu được stretch. Hãy dồn chi tiết quan trọng về góc hoặc một vùng cạnh có thể lặp/kéo giãn an toàn.

### 6. Khuyến nghị kỹ thuật cho asset

- Định dạng: PNG
- Khuyến nghị: có alpha channel
- Khuyến nghị: center transparent hoàn toàn
- Khuyến nghị: khung có độ dày viền rõ ràng, dễ đo theo pixel
- Khuyến nghị: dùng asset vuông hoặc gần vuông để preset dễ cân hơn, dù không bắt buộc

## Checklist bàn giao

- Center là vùng rỗng để artwork hiển thị
- File PNG có alpha
- Có số đo độ dày viền ở 4 cạnh
- Có ghi chú phần nào của cạnh có thể kéo dài
- Góc không phụ thuộc vào việc bị scale
- Không có hiệu ứng quan trọng nằm trong center
- Có mô tả độ dày mặt khung mong muốn ngoài đời

## Ghi chú cho team

- Hành vi renderer hiện tại nằm ở `riz-app-v2/screens/create-flat2d-framing/lib/nine-slice.ts` và `riz-app-v2/screens/create-flat2d-framing/lib/framed-artwork-layout.ts`.
- Spec sản phẩm chính của tính năng nằm ở `docs/SPEC-framing-flat2d-v2.vi.md`.
- Nếu sau này muốn cho frame có hiệu ứng phủ một phần lên artwork, cần mở rộng renderer thay vì chỉ sửa asset.
