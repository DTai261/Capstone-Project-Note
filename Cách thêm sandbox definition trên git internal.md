
### 1. Dùng sshuttle để forward trafic đến subnet 10.1.2.0/24 
![](attachments/Pasted%20image%2020230702095600.png)

### 2. SSH đến máy ảo
file gcp.key đã đc đẩy lên trello.
```bash
ssh ubuntu@34.136.10.86 -i gcp_2.key
```

### 3. SSH đến openstack qua user vagrant
```
user:password
vagrant:vagrant
```
![](attachments/Pasted%20image%2020230702095902.png)

tiếp đến`sudo su` để switch sang user root -> cd đến /root -> chạy binary k9s:
![](attachments/Pasted%20image%2020230702100059.png)

### 4. Một số thao tác cơ bả trong k9s
- nút điều hướng để di chuyển chọn pod :)
- `enter` để chọn vào pod
- `s` để get shell vào 1 instance trong pod, ví dụ như trong pod `git-internal-...` sẽ có 2 instance thì phải chọn pod trước rồi chọn instance để get shell
- `esc` để back lại
- `ctrl + C` để thoát
![](attachments/Pasted%20image%2020230702100857.png)

### 5. Add sandbox definition vào git internal

get shell `git-internal-ssh`
![](attachments/Pasted%20image%2020230702101012.png)

cd tới `repos`. Như mn cũng đã thấy thì kypo lite chỉ cho phép thêm sand box definition từ git internal chứ k kéo thẳng từ trên web xuống đc, và nó có yêu cầu là tất cả phải ở trong directory `repos`.
Hiện tại thì mn đang kéo git về khá là lộn xộn như này:
![](attachments/Pasted%20image%2020230702101123.png)

Nên tôi muốn mn từ giờ tất cả lab mình tạo kéo về `/repos/Labs` và đặt tên repository theo định dạng `<Lab thứ mấy>-<Tên lab theo tên trên syllabus>`
![](attachments/Pasted%20image%2020230702101705.png)

*Clone repo từ git về:*
```bash
git clone -q --bare <link-to-git-repo>
```