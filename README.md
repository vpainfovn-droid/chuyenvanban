# chuyenvanban

Web app chép lời tiếng Việt/Anh bằng **OpenAI Whisper (`whisper-1`)**, FastAPI, SQLite và giao diện HTML/CSS/JavaScript. Không cần Node.js. FFmpeg đọc và chia file. Backend giữ key, xử lý tuần tự một tác vụ và lưu trạng thái ngoài yêu cầu HTTP.

## Chạy trên máy tính

1. Cài Python 3.12 trở lên từ python.org. Cài FFmpeg (cần cả `ffmpeg` và `ffprobe` trong PATH). Windows có thể dùng `winget install Gyan.FFmpeg`, macOS dùng `brew install ffmpeg`, Ubuntu dùng `sudo apt install ffmpeg`.
2. Mở terminal trong thư mục `chuyenvanban`.
3. Tạo môi trường và cài thư viện:

```sh
python -m venv .venv
# macOS/Linux:
source .venv/bin/activate
# Windows PowerShell dùng: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

4. Sao chép `.env.example` thành `.env`. Thêm key OpenAI của bạn vào `OPENAI_API_KEY`. Key phải có quyền dùng Audio API và tài khoản có hạn mức thanh toán. Không gửi key vào chat hoặc commit file `.env`.
5. Chạy ứng dụng (Uvicorn hỗ trợ nạp `.env` khi cài python-dotenv):

```sh
pip install python-dotenv==1.2.2
uvicorn app.main:app --env-file .env --host 127.0.0.1 --port 8000
```

6. Mở trình duyệt ở `http://127.0.0.1:8000`. Chọn file, chọn ngôn ngữ và nhấn **Bắt đầu chép lời**. Nhấn mốc thời gian để nghe lại. Sửa từng đoạn, lưu, rồi xuất TXT/DOCX/SRT. Dịch hoặc tạo bản dễ đọc chỉ chạy khi bấm nút riêng.

Nếu chưa có key, ứng dụng vẫn mở và nhận file, sau đó hiện lỗi cấu hình rõ ràng. Thêm key, khởi động lại backend rồi bấm **Thử lại**. Không có nhận dạng giả thay thế.

## Dữ liệu và chi phí

- File gốc lưu trong `data/<mã tác vụ>/original.*`; trạng thái, bản chép và bản bổ sung lưu trong `data/jobs.sqlite3`. Có thể đổi thư mục qua `DATA_DIR`.
- Thời hạn mặc định 24 giờ kể từ tải lên. Worker xóa file và bản ghi hết hạn ở mỗi vòng xử lý; nếu một file dài đang xử lý, việc dọn dẹp có thể trễ tới khi file đó kết thúc. Khi máy chủ tắt, dữ liệu được dọn khi chạy lại. SQLite có thể còn dấu vết bản ghi đã xóa trong trang đĩa; không tuyên bố xóa an toàn ở cấp thiết bị hoặc bản sao lưu.
- Âm thanh gửi qua HTTPS tới `api.openai.com/v1/audio/transcriptions`. Khi bạn chọn dịch/bản dễ đọc, văn bản gửi tới OpenAI Chat Completions (`gpt-4o-mini`). Chính sách lưu của nhà cung cấp khác với thời hạn cục bộ; xem chính sách dữ liệu OpenAI trước khi tải dữ liệu nhạy cảm.
- Mặc định tối đa 200 MB, 240 phút. Giá tham khảo Whisper 0,006 USD/phút, không phải báo giá cam kết. Phần chồng lấn, thử lại và các thao tác văn bản có thể phát sinh phí thêm. Kiểm tra giá hiện hành tại OpenAI.
- URL tác vụ chứa mã ngẫu nhiên, là quyền truy cập dữ liệu. Giao diện nhớ tác vụ gần nhất trong localStorage của trình duyệt. Không chia sẻ mã/URL. Không có tài khoản hoặc đăng nhập.

## Độ chính xác và giới hạn hiện tại

- Không tóm tắt bản chép. Yêu cầu dịch vụ giữ từ đệm/câu lặp nhưng mô hình vẫn có thể bỏ, nhận nhầm hoặc tạo lời sai; không thể cam kết 100%. Cần nghe lại.
- Chia 10 phút, chồng lấn 5 giây mỗi bên. Chọn từ theo mốc thời gian tuyệt đối ở ranh giới 10 phút để không giữ hai bản sao cùng vùng âm thanh. Dịch vụ có thể lệch mốc và bỏ/nhân đôi từ ở ranh giới; cần kiểm tra file dài thực tế.
- Whisper không trả nhãn người nói hoặc độ chắc chắn từng từ trong chế độ đang dùng. Vì vậy chưa có tự động phân biệt/đổi tên người nói hoặc tự động xác định mọi đoạn không nghe rõ. Nút **Đánh dấu không nghe rõ** thêm `[không nghe rõ – phút:giây]` vào đoạn đang sửa. Không đoán bằng mô hình thứ hai.
- Xuất SRT theo các đoạn tối đa khoảng 8 giây/24 từ; thời gian được giữ khi sửa văn bản. Bản dịch/dễ đọc tách riêng, chưa có xuất DOCX/SRT cho bản bổ sung. Có thể chọn và sao chép bản bổ sung.
- Chép lời chạy nền, phục hồi tác vụ đang xử lý về hàng đợi khi khởi động lại. Đoạn hoàn thành chỉ lưu trong bộ nhớ đến khi cả file xong; khi lỗi/khởi động lại, thử lại toàn file có thể tốn phí lần nữa.
- Dịch/chỉnh dấu câu tối đa 40.000 ký tự, là yêu cầu HTTP riêng (không phải hàng đợi nền); văn bản lớn hơn cần chia nhỏ. Không tự cắt để tạo kết quả thiếu.
- Một worker, một máy chủ, một tiến trình Uvicorn. Không chạy nhiều worker trên cùng SQLite. Đây là ứng dụng dùng cá nhân; cần đăng nhập, giới hạn tần suất, quota, HTTPS và worker riêng trước khi mở công khai.

## Kiểm tra

```sh
python -m pytest -q
```

Tests dùng WAV im lặng sinh tại thời điểm chạy để kiểm tra upload/FFprobe, lỗi thiếu key, xóa; bản chép fixture riêng để kiểm tra sửa/xuất DOCX/TXT/SRT; kiểm tra ghép chồng lấn. **Đây không phải kiểm chứng nhận dạng thực tế.** Để kiểm tra thật: cấu hình key, tải bản ghi tiếng Việt đã biết nội dung (gồm từ đệm và lặp), kiểm tra nghe lại và toàn bộ văn bản; kiểm tra thêm file trên 10 phút ở ranh giới; sửa/lưu/tải lại trang/xuất cả ba loại. Xác nhận phút:giây của SRT bằng trình phát phụ đề.

## Triển khai

Dùng máy Linux hoặc container có FFmpeg, volume bền vững cho `data/`, biến môi trường `OPENAI_API_KEY` và HTTPS qua reverse proxy. Chạy `uvicorn app.main:app --host 0.0.0.0 --port 8000` **một worker**. Đặt ứng dụng sau xác thực của reverse proxy/VPN cho sử dụng cá nhân; không mở trực tiếp lên Internet vì chưa có đăng nhập và quota. Reverse proxy cần cho upload 200 MB và timeout phù hợp cho dịch văn bản. Không đưa `.env` vào image.

```sh
docker build -t chuyenvanban .
docker run --env-file .env -p 127.0.0.1:8000:8000 -v chuyenvanban-data:/app/data chuyenvanban
```

Không có đường dẫn preview công khai được cấp trong môi trường onboarding này. Có thể dùng hướng dẫn chạy máy tính ở trên để truy cập bằng trình duyệt.
