Bài này có 1 điểm mấu chốt là cần upload được file webshell lên là thành công, nhưng lại có tận 2 lần filter
1. WAF Openresty whitelist chúng ta chỉ được phép upload các file `jpg`, `jpeg`, `png`, `gif`
<img width="1295" height="833" alt="image" src="https://github.com/user-attachments/assets/615f462b-d369-44de-a222-76b984bd0143" />

2. Đoạn xử lý upload file cũng đã check xem extension có trong allowedExtension hay không
<img width="1060" height="829" alt="image" src="https://github.com/user-attachments/assets/0c12a765-a263-4967-9601-222e8fd676d7" />

Ngoài ra trong `nginx.conf` đã chặn chúng ta truy cập vào `/upload`, nơi mà các file được upload lên.

Vậy có 2 câu hỏi chính:
- Làm sao để upload 1 webshell lên?
- Rồi upload xong thì truy cập webshell kiểu gì?


Tất nhiên là các filter đều có vấn đề như sau:
### Bypass OpenResty Lua multipart parsers
Lấy ý tưởng từ bài viết này https://blog.sicuranext.com/breaking-down-multipart-parsers-validation-bypass/#bypass-openresty-lua-multipart-parsers, phần rule của waf viết bằng Lua khá là giống với mẫu trong bài viết.
<img width="1374" height="611" alt="image" src="https://github.com/user-attachments/assets/fbf48adf-39f8-4ebe-ae5d-44e5dba9e878" />

Chỉ cần duplicate biến filename trong multipart data, chúng ta sẽ bypass được vì
- Lua resty multipart lib nó lấy biến `filename` đầu tiên 
- Còn trình biên dịch của PHP sẽ lấy biến `filename` thứ 2
-> Chỉ cần thêm 1 biến `filename` nữa vào là được

### Bypass Upload file check and 403 forbidden in upload folder

```php=
$target_dir = "upload/" . $_SESSION['username'] . "/";
if (!is_dir($target_dir)) {
  mkdir($target_dir, 0777, true);
}

$allowedExtensions = ['jpg', 'jpeg', 'png', 'gif'];

if (isset($_FILES['avatar']['full_path']) && !empty($_FILES['avatar']['full_path'])) {
  $fileName = $_FILES['avatar']['full_path'];
} else {
  $fileName = $_FILES['avatar']['name'];
}
// Check if file already exists
if (file_exists($target_dir . $fileName)) {
  exit("Sorry, file already exists.");
}

$fileExt = explode('.', basename($fileName))[1];

$target_file = $target_dir . $fileName;

if (!in_array($fileExt, $allowedExtensions)) {
  exit("Sorry, your file type is not allowed.");
}
```

Đoạn xử lí này nó chỉ check file extension là `$fileExt = explode('.', basename($fileName))[1];` cho nên chúng ta chỉ cần bypass bằng cách `poc.png.php` do đoạn check chỉ kiểm tra phần tử thứ 2 trong mảng explode với separater là dấu `.`.

Ngoài ra ở đây chúng ta còn có thể upload ra ngoài folder `/upload/` do đoạn xử lí như sau 
```php=
if (isset($_FILES['avatar']['full_path']) && !empty($_FILES['avatar']['full_path'])) {
  $fileName = $_FILES['avatar']['full_path'];
} else {
  $fileName = $_FILES['avatar']['name'];
}
```

Nếu trong file tải lên có key là `full_path` nó sẽ giữ nguyên các dấu như kiểu là `../../` bởi vì 1 tính năng mới trong PHP 8.1 https://php.watch/versions/8.1/$_FILES-full-path.

Tính năng này cho phép giữ nguyên đường dẫn của file tải lên. Và khi move_uploaded_file là ta có thể path traversal được thôi.

Upload ra ngoài `/var/www/html` rồi truy cập webshell là thành công.
