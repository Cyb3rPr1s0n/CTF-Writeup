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


