---
title: "Write-up No Hack No CTF 2026"
date: 2026-07-07 14:11:00 +0700
categories: [Capture Flag, Write-up No Hack No CTF 2026]
tags: [ctf, wu, crypto, osint]
math: true
image:
  path: /assets/img/posts/no-hack-no-ctf-2026-cover.jpg
  alt: No Hack No CTF 2026
---

Đang trong thời gian nghỉ hè nên là mình có đánh giải No Hack No CTF 2026 cùng với một người bạn nữa để giải trí và học hỏi thêm thì dưới đây là write up các bài mình đã giải được.

## newbie-crypto
![alt text](/assets/img/posts/no-hack-no-ctf-1.png)

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
![alt text](/assets/img/posts/no-hack-no-ctf-2.png)

Đầu tiên thì bài này là một bài osint, mình phải tìm được 2 thứ là:

- Ngày ảnh được chụp
- IATA flight number của chuyến bay

### Date the photo 
Đầu tiên thì trong đề có ghi là "That day in Japan marked the end of Golden Week" thì đây là một ngày tuần lễ vàng người dân Japan được nghỉ để nghĩ dưỡng các thứ. Ngày này rơi vào cuối tháng 4 đến đầu tháng 5. 

![alt text](/assets/img/posts/Final-Boarding-1.png)

Nhưng trong bài này khả năng cao là đầu tháng 5 vì nó ghi là "End of Golden Week".

Để biết chính xác thì mình xem metadata của tấm hình đề cho 

![alt text](/assets/img/posts/Final-Boarding-2.png)

Ta thấy +09:00 là múi giờ Nhật Bản, nên thời gian này là giờ địa phương tại Nhật và thời gian này cũng khớp so với thời gian Golden Week thực tế.

=> 2026-05-05

### IATA flight number 

Nhìn vào tấm hình thì có thể thấy đuôi máy bay là của hãng Jetstar Japan số hiệu A320 và bên cạnh đó là registration của aircraft là JA06JJ.

![alt text](/assets/img/posts/Final-Boarding-3.png)

Đề nói người chụp đang đứng trước boarding gate tại Nhật mà ở Nhật có hơn khoảng 55 sân bay nên đầu tiên phải xác định được sân bay nằm ở ga nào thì với các thông tin như mã số, loại máy bay thì mình search được lịch bay của chiếc máy bay này. 

![alt text](/assets/img/posts/Final-Boarding-4.png)

Nhìn vào bảng thì mình thấy cái khớp nhất là GK55 vì 14:49 JST tức khoảng 40 phút trước khi bay thì người này đã đứng chụp tấm ảnh này. 

=> GK55

Vậy từ hai dữ kiện trên thì flag cuối sẽ là: ```NHNC{20260505_GK55}```

### Kira-Notes

### LemonShelf

### Camel rider


### Homework


### BabyZKP

### TEARoam
