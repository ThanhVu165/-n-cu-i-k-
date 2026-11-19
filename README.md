# Automata Lab — DFA Equivalence Checker (T12)

Ứng dụng web minh họa cách kiểm tra hai DFA có nhận cùng một ngôn ngữ hay không. Phần chính không gọi thư viện automata: chương trình tự chuẩn hóa bảng chữ cái, mô hình sink ẩn, duyệt tích trạng thái bằng BFS/DFS, dựng phản ví dụ và mô phỏng kiểm chứng. Giao diện phát lại chính event do thuật toán ghi nhận.

## Chạy ứng dụng

Yêu cầu Node.js 20 trở lên và npm.

```bash
npm install
npm run dev
```

Mở địa chỉ local Vite hiển thị trong terminal. Tạo bản build tĩnh bằng `npm run build`; chạy toàn bộ kiểm thử bằng `npm test`.

## Dùng nhanh

1. Chọn một trong năm kịch bản demo hoặc mở “Chỉnh sửa định nghĩa DFA” ở từng máy.
2. Khai báo trạng thái, token của bảng chữ cái, trạng thái đầu/chấp nhận và các dòng `nguồn, ký hiệu → đích`.
3. Nhấn **So sánh ngôn ngữ**. Có thể chọn BFS hay DFS, phát/tạm dừng, đi từng bước, kéo thanh timeline và đổi tốc độ.
4. Với kết quả khác nhau, xem token phản ví dụ và trace được mô phỏng độc lập trên cả hai DFA. Với kết quả tương đương, đồ thị tích kết thúc khi frontier cạn.
5. Nhập/xuất từng máy dưới dạng JSON (tệp tối đa 1 MB). Bộ nhập hỗ trợ JSON DFA thuần và envelope phiên bản `t12-dfa/v1`; schema lạ, khóa JSON trùng và dữ liệu sai bị từ chối.

## Mô hình và thuật toán

DFA được biểu diễn bằng bộ `(Q, Σ, δ, q₀, F)`. Phép so sánh dùng `Σ = ΣA ∪ ΣB`. Một chuyển không được khai báo, kể cả khi token chỉ có ở DFA kia, được hiểu là chuyển vào sink bác bỏ. Sink tự lặp trên mọi token. Quy ước này không thay đổi ngôn ngữ mà DFA từng khai báo, đồng thời giúp hai máy dùng cùng một bảng chữ cái.

Thuật toán trung tâm tạo **tích đạt được** từ cặp đầu `(q₀A, q₀B)`. Với token `a`, cạnh tích là `(p,q) → (δA(p,a), δB(q,a))`. Cặp mismatch thỏa `(p ∈ FA) XOR (q ∈ FB)`, tương đương một từ phân biệt hai ngôn ngữ. Nếu gặp mismatch ở cặp đầu, witness là `ε`. Nếu không, BFS dùng hàng đợi FIFO và lưu predecessor cùng token cho mỗi cặp mới. BFS thăm các cặp theo lớp độ dài; do đó witness mismatch đầu tiên có số token nhỏ nhất. Nếu duyệt hết các cặp đạt được mà không thấy mismatch thì hai ngôn ngữ bằng nhau.

DFS chạy trên cùng ngữ nghĩa tích để đối chiếu việc mismatch có đạt được hay không. DFS **không** bảo đảm phản ví dụ ngắn nhất. Mỗi witness được phát ra sẽ được `simulate()` chạy lại trên hai DFA đầu vào; giao diện chỉ đánh dấu xác minh nếu đúng một máy chấp nhận.

Với `nA = |QA|`, `nB = |QB|`, `k = |ΣA ∪ ΣB|`, lõi tìm kiếm có worst case `O(k · nA · nB)` thời gian và `O(nA · nB)` bộ nhớ (cộng chi phí xử lý token/chuỗi trạng thái). Sink được tính vào kích thước máy đã chuẩn hóa khi hiển thị cận cặp lý thuyết. Cài đặt duyệt lười, nên chỉ vật chất hóa các cặp đi tới được; BFS có thể dừng sớm khi gặp mismatch. Để animation có thể phát lại, chương trình cũng ghi event/cạnh đã xét, nên bộ nhớ toàn phiên có thể tăng tới `O(k · nA · nB)` ngoài trạng thái lõi.

## Đánh giá lần chạy

- `visitedPairCount`: số cặp tích duy nhất được tạo trong lần tìm kiếm.
- `pairStateUpperBound`: tích số trạng thái sau khi thêm sink cần thiết; cận lý thuyết, không phải số cặp thực tế.
- `transitionAttempts`: số lần xét một token từ một cặp đang mở rộng.
- `elapsedMs`: một lần đo bằng `performance.now()`; phụ thuộc máy, trình duyệt, tải và độ lớn đồ thị. Không dùng riêng lần đo nhỏ để khẳng định thuật toán nhanh hơn.
- `witness length`: số token, khác với số ký tự nếu nhãn token dài nhiều ký tự.

Màn kết quả cho xem trace trạng thái của cả hai máy, kết luận, xác minh phản ví dụ và số liệu BFS/DFS. DFS có thể tìm witness khác hoặc dài hơn do thứ tự duyệt. Kết quả so sánh thời gian một lần chỉ để quan sát; đánh giá hiệu năng nghiêm túc cần nhiều lượt, warm-up và kiểm soát môi trường.

## Kịch bản và kiểm thử

Ứng dụng có ví dụ cho DFA tương đương dù khác tên/partiality, mismatch `ε`, phản ví dụ dài hai token, chuyển thiếu và bảng chữ cái không bằng nhau. Kiểm thử gồm input sai/thiếu/trùng, JSON hỏng/schema lạ, cặp không đạt được, ngôn ngữ rỗng, mô phỏng sink, BFS shortest witness, và 120 cặp DFA nhỏ sinh xác định để đối chiếu BFS/DFS cùng mô phỏng witness.

```bash
npm test
npm run build
```

## Giới hạn hiện tại

- Chỉ quyết định DFA; không chuyển đổi NFA, biểu thức chính quy, CFG, PDA hay Turing machine.
- Trạng thái và token nhập dưới dạng danh sách phân tách bằng dấu phẩy; nhãn không được rỗng, trùng, hoặc có khoảng trắng đệm.
- Mỗi DFA giới hạn 40 trạng thái và 16 token bảng chữ cái; dữ liệu vượt giới hạn bị báo lỗi nguyên trạng, không bị cắt bớt. Đây là giới hạn tương tác của ứng dụng, không phải giới hạn lý thuyết của DFA.
- Bộ nhập JSON giới hạn tệp ở 1 MB để ngăn xử lý chuỗi/JSON quá lớn trong tab trình duyệt.
- SVG hiển thị tối đa 300 nút và 600 cạnh, giữ ưu tiên cặp đầu/hiện tại/mismatch/frontier; frontier tối đa 24 nhãn. Phát lại tối đa 299 bước đầu và event kết thúc thật; các bước giữa được tóm tắt nếu cần. Thuật toán và thống kê vẫn duyệt chính xác trên toàn bộ input hợp lệ, còn phần hình/animation rút gọn có thông báo rõ.
- Lưu event phục vụ animation làm bộ nhớ ứng dụng lớn hơn bộ nhớ lõi BFS/DFS; không nên dùng giao diện này cho máy cỡ lớn thay cho công cụ phân tích chuyên dụng.
- DFS chỉ là đối chiếu, không phải benchmark độc lập vì hai thuật toán dùng chung mô hình và runtime.
- Dữ liệu người dùng chỉ tồn tại trong phiên trang; không có backend hoặc lưu trữ tài khoản.

## Ranh giới thư viện và phần tự cài

| Thành phần | Nguồn |
| --- | --- |
| React, React DOM | Thư viện giao diện; render component và quản lý state UI |
| Vite, TypeScript | Dev server, bundler và kiểm tra kiểu |
| Vitest, Testing Library, jsdom | Kiểm thử |
| DFA validator, union alphabet, sink semantics, product graph transitions | Nhóm tự triển khai |
| BFS tìm shortest witness, DFS emptiness cross-check, predecessor reconstruction | Nhóm tự triển khai |
| Witness simulation, event replay, counters và timing | Nhóm tự triển khai |
| SVG, CSS, browser `performance.now()` | API/định dạng nền tảng trình duyệt; không có thư viện automata/đồ thị |

## Giải thích mã nguồn

- `src/automata/`: model, validator, chuẩn hóa, BFS, DFS, simulation, serialization và replay.
- `src/examples/`: bộ DFA demo nhỏ, cố định.
- `src/components/`: biên tập, SVG, playback, kết quả, đối chiếu và liên hệ lý thuyết.
- `docs/superpowers/specs/` và `docs/superpowers/plans/`: quyết định thiết kế và kế hoạch implementation.

Xem [AI_USAGE.md](./AI_USAGE.md) để ghi nhận minh bạch phần hỗ trợ bằng AI và checklist nhóm cần tự xác minh trước khi viết báo cáo.

