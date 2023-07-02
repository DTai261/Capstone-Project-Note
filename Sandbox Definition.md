Cấu trúc một git repo để tại sanbox definition:
![](attachments/Pasted%20image%2020230626222757.png)
Tổng quan một chút thì để cấu hình Ansible thì ta sử dụng các file YAML (Ain't Markup Language). Ngoài ra ta có thể dùng các định dạng như JSON, XML để cấu hình nhưng trong phạm vi làm đồ án cũng như sự tiện dụng và linh hoạt hơn của YAML nên t chỉ đề cập đến YAML trong phần giới thiệu này. 
Đi sâu hơn vào các thành phần có trong một training definition của kypo. Thì như mn thấy ở repo [kypo-crp-demo-training](https://gitlab.ics.muni.cz/muni-kypo-crp/prototypes-and-examples/sandbox-definitions/kypo-crp-demo-training/-/tree/master/) có nhưng file như variables.yml, traning.json, topology.yml và folder provisioning,...
# 1. variables.yml
```yml
telnet_port:
    type: port
    min: 1500
alice_flag:
    type: text
root_flag:
    type: text
```

Đúng như tên gọi của file, ở đây cta khởi tạo kiểu giá trị cho các biến sẽ sử dụng trong quá trình config. Ví dụ như: 
- `telnet_port` có kiểu dữ liệu là port -> biến này chỉ định một port có giá trị nhỏ nhất khi set là 1500, 
- `alice_flag` có kiểu giữ liệu là text -> biến này ta có thể khai báo bất kỳ một string nào.

# 2. traning.json
```json
{
  "title" : "KYPO Cyber Range Training Platform - Demo Content (terminal)",
  "description" : "The demo contains all types of levels to demonstrate the training capabilities of the KYPO Cyber Range Platform.",
  "prerequisites" : [ ],
  "outcomes" : [ "Scanning ports with nmap", "Password guessing with hydra at telnet", "Privilege escalation using misconfigured sudo" ],
  "state" : "UNRELEASED",
  "show_stepper_bar" : true,
  "levels" : [ {
    "title" : "Info",
    "level_type" : "INFO_LEVEL",
    "order" : 0,
    "estimated_duration" : 0,
    "minimal_possible_solve_time" : null,
    "content" : "# ... Content ..."
  }, {
    "title" : "Get Access",
    "level_type" : "ACCESS_LEVEL",
    "order" : 1,
    "estimated_duration" : 0,
    "minimal_possible_solve_time" : null,
    "passkey" : "start",
    "cloud_content" : "... another content ...",
    "local_content" : "This sandbox has not been tested for use with local sandbox."
  }
  ***
```

Ở file này cta sẽ tạo nội dung cho các Phases traning (phần này không cần quá quan tâm vì có thể thực hiện trên GUI web như hình bên dưới):
![](attachments/Pasted%20image%2020230626224653.png)

# 3. topology.tml
```yaml
name: kypo-crp-demo-training

hosts:
  - name: server
    base_box: 
      image: ubuntu-focal-x86_64
      man_user: ubuntu
    flavor: standard.small

  - name: client
    base_box: 
      image: ubuntu-focal-x86_64
      man_user: ubuntu
    flavor: standard.small

routers:
  - name: router
    base_box:
      image: debian-9-x86_64
      man_user: debian
    flavor: standard.small
***
groups: []

```

`topology.yaml` được sử dụng để xác định cấu trúc liên kết mạng và các đặc điểm của các hosts, router, switch và nhóm các thiết lập hệ thống hoặc cơ sở hạ tầng. Trong nhóm các thiết lập này đc chia nhỏ: 
- **name**: thiết lập tên cho network topology (ở đây là "kypo-crp-demo-training")
- **hosts**: Define các hosts trong network, ở ví dụ này 2 hosts đc define *server* và *client*. Mỗi hosts lại có những thuộc tính sau:
	- **name**: tên hosts
	- **base_box**: Define base image cho host, ở cả *server* và *client* trong vd này đều sử dụng image `ubuntu-focal-x86_64` và managerment user set là *ubuntu*
- *flavor*: chỉ định config hardware cho host, ở đây thì cả 2 máy đều set là "standard.small"

*Note 1:* đây là những flavor mà hiện tại openstack của project đang hỗ trợ, mn khi tạo các máy ảo nhớ tham khảo để chọn cấu hình phù hợp:
![](attachments/Pasted%20image%2020230628150149.png)

# 4. provisioning 
Cấu trúc của các instance như môi trường, tạo user, install packages,... Sandbox Provisioning phải chỉ ra được cách kết nối tới các instance: username, ssh key
![](attachments/Pasted%20image%2020230627004942.png)

## 4.1 requirements.yml
```yaml
- name: kypo-user-access
  src: https://gitlab.ics.muni.cz/muni-kypo-crp/backend-python/ansible-networking-stage/kypo-user-access.git
  scm: git
  version: 470b8b91

- name: kypo-disable-qxl
  src: https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/disable-qxl.git
  scm: git
  version: 7981c7c4

- name: sandbox-logging
  src: https://gitlab.ics.muni.cz/muni-kypo/ansible-roles/sandbox-logging.git
  scm: git
  version: 5431051c
```
file `requirements.yml` chỉ định một list các ansible roles từ bên ngoài mà playbook/project cần. Vì phần này t chưa tìm hiểu kỹ nên tạm thời chưa động đến và mặc định để nó install trước khi chạy playbook. :3

## 4.2 playbook.yml
```yaml
- name: disable qxl
  hosts:
      - routers
      - hosts
  gather_facts: yes
  become: yes
  tasks:
      - include_role:
            name: kypo-disable-qxl
        when: ansible_os_family == 'Debian'

- name: set up server
  hosts: server
  become: yes
  roles:
    - name: server
      telnet_port: "{{ telnet_port }}"
      flag: "{{ alice_flag }}"
      flag_2: "{{ root_flag }}"

- name: set up client
  hosts: client
  become: yes
  roles:
    - name: kypo-user-access
      kypo_user_access_username: user
      kypo_user_access_password: Password123
      kypo_user_access_sudo: True
    - name: client

- name: set up command logging
  hosts: hosts
  become: yes
  roles:
    - role: sandbox-logging
      slf_destination_port: '{% if hostvars["man"] is defined -%} 514 {%- else -%} 515 {%- endif %}'

```

