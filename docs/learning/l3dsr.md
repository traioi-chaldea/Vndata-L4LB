# Layer 3 Load Balancing

**L3DSR** được Yahoo công bố opensource tại [Github Link](https://github.com/yahoo/l3dsr).

## Advantage

So với L2DSR thì **L3DSR** có nhiều cải tiến hơn như:
* Backend có thể thấy được Client IP.
* Do sử dụng Layer 3 để tương tác với Backend nên không còn bị giới hạn vị trí địa lý của các Backend nằm phía sau Load Balancer.
* Sử dụng DSCP để Backend xác định VIP tương ứng từ Load Balancer, giúp Load Balancer phân loại được packet theo Services.

## How DSCP work in L3DSR

### What is DSCP?

* **DSCP** là viết tắt của **D**ifferentiated **S**ervices **C**ode **P**oint.
* **DSCP** được sử dụng trong việc đánh dấu packet để mô tả "tính đặc biệt" của packet, được sử dụng trong QoS để phân loại packet khi đi qua.
* Trong IP Header có 1 field là ToS (Type of Service) gồm 8 bits. Trong đó:
  * **DSCP** sử dụng 6 bits đầu trong việc phân loại packet. 
  * 2 bits sau không được sử dụng.

### DSCP in L3DSR

* **DSCP** được sử dụng để mark packet trong các giải pháp ToS, QoS, COS nhưng đây lại là field mà Yahoo không sử dụng => Yahoo có 6 bits không sử dụng, đó chính là 6 bits của **DSCP**.
* Các packet mang **DSCP** sẽ được Yahoo map với VIP tương ứng bằng module `dscp` trong `iptables` (được cấu hình trên Backend). Ví dụ:
```
# iptables -t mangle -A INPUT -m dscp --dscp 1 -j DADDR --set-daddr=192.168.1.100
# iptables -t mangle -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DADDR      all  --  anywhere             anywhere             DSCP match 0x01DADDR set 192.168.1.100
```

## Dataflow 

![l3dsr-01](/docs/img/l3dsr-01.png)

* VIP sẽ được mapping như sau:

| DSCP | Service | VIP |
| --- | --- | --- |
| 0x01 | Web | 192.168.1.100 |
| 0x02 | FTP | 192.168.1.200 |

* **(1):** Client (192.168.0.100) muốn truy cập đến các Backends phải thông qua Load Balancer với DSCP lúc này chưa được set nên có giá trị là 0x00.
  * Trường hợp Client truy cập Web nên sử dụng IP 192.168.1.100.
    * **Source IP:** 192.168.0.100
    * **Dest IP:** 192.168.1.100
    * **DSCP:** 0x00
  * Trường hợp Client truy cập FTP nên sử dụng IP 192.168.1.200.
    * **Source IP:** 192.168.0.100
    * **Dest IP:** 192.168.1.200
    * **DSCP:** 0x00
* Lúc này, Load Balancer sẽ xác định Backend cần forward packet về thông qua 2 bước:
  * **Health check:** Tùy theo dịch vụ mà Load Balancer sẽ xác định Backend "còn sống" hay "đã chết", ví dụ đối với Web Service thì Load Balancer sẽ gửi gói tin HTTP cùng với DSCP tương ứng (0x01), nếu Backend "còn sống" thì sẽ response với status 200 cho Load Balancer.
  * Sau khi xác định được Backend "còn sống" nhiều hơn 2 thì Load Balancer sẽ sử dụng thuật toán tương ứng (Round Robin, Least Conn, ..) để tìm ra Backend phù hợp.
* **(2):** Load Balancer thay đổi DSCP tương ứng với Serivce mà Client truy cập trong header và forward packet về Backend tương ứng với header:
  * Trường hợp truy cập Web:
    * **Source IP:** 192.168.0.100
    * **Dest IP:** 10.0.0.2 (hoặc 10.0.0.3)
    * **DSCP:** 0x01
  * Trường hợp truy cập FTP:
    * **Source IP:** 192.168.0.100
    * **Dest IP:** 10.0.0.2 (hoặc 10.0.0.3)
    * **DSCP:** 0x02
* **(3):** Lúc này, Backend sẽ response lại packet trực tiếp cho Client mà không cần qua Load Balancer. Backend sẽ dựa vào cấu hình routing và DSCP (ví dụ như `iptables`) để thay đổi response header tương ứng với Service mà Client truy cập.
  * Trường hợp truy cập Web:
    * **Source IP:** 192.168.1.100
    * **Dest IP:** 192.168.0.100
  * Trường hợp truy cập FTP:
    * **Source IP:** 192.168.1.200
    * **Dest IP:** 192.168.0.100