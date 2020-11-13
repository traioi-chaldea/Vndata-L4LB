# Layer 2 DSR

## Dataflow

![l2dsr-01](/docs/img/l2dsr-01.png)

**Request** là khi Client (192.168.0.100) muốn gửi 1 packet đến Backend (10.0.0.2) thì phải gửi thông qua Load Balancer (192.168.1.100).
* **(1)**: Lúc này packet sẽ đi qua Load Balancer (192.168.100). Header lúc này sẽ là:
	* **Source IP:** 192.168.0.100
	* **Dest IP:** 192.168.1.100
* **(2)**: Load Balancer sẽ xác định Backend đích bằng cách gửi broadcast ra toàn bộ LAN để tìm ra Backend nào đang hoạt động. Sau đó, sẽ đồng thời làm 2 việc là thay đổi địa chỉ MAC đích từ header và forward request của Client đó đến Backend đích thông qua **Pri NIC**. Header lúc này sẽ là:
	* **Source IP:** 192.168.0.100
	* **Dest IP:** 192.168.1.100
	* **Dest MAC:** BBBB
	* *Trên Backend phải cấu hình loopback IP (VIP) tương ứng (192.168.1.100) để Backend không gửi ARP về Load Balancer.*

**Response** là lúc Backend trả lời lại cho Client.
* **(3)**: Backend trả lời lại cho Client với header:
	* **Source IP:** 10.0.0.1
	* **Dest IP:** 192.168.0.100
	* **Dest MAC:** Default gateway MAC.

## Disadvantage

* Backend không thấy được IP của Client.
* Các Backend server chỉ hoạt động được chung 1 L2 network.
	* Location bị hạn chế vì các Backend phải nằm chung 1 rack hoặc datacenter với Load Balancer.
* Các hạn chế liên quan đến L3, L4 như load balancing theo port hoặc IP.
