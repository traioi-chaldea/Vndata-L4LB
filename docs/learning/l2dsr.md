# Layer 2 DSR

## Dataflow

![l2dsr-01](/docs/img/l2dsr-01.png)

**Request** là khi Client (192.168.0.100) gửi 1 packet đến Backend (192.168.0.200).
* **(1)**: Lúc này packet sẽ đi qua Load Balancer (10.0.0.1) trước. Header lúc này sẽ là:
	* **Source IP:** 192.168.0.100
	* **Dest IP:** 10.0.0.1
	* **Dest MAC:** AAAA
* **(2)**: Load Balancer sẽ xác định Backend đích. Sau đó, sẽ đồng thời làm 2 việc là thay đổi địa chỉ MAC đích từ header và forward request của Vlient đó đến Backend đích dựa vào địa chỉ MAC đã thay đổi. Header lúc này sẽ là:
	* **Source IP:** 192.168.0.100
	* **Dest IP:** 10.0.0.1
	* **Dest MAC:** BBBB
	* *Trên Backend phải cấu hình loopback IP tương ứng (10.0.0.1) để nhận được request từ Load Balancer.*

**Response** là lúc Backend trả lời lại cho Client.
* **(3)**: Backend trả lời lại cho Client với header:
	* **Source IP:** 10.0.0.1
	* **Dest IP:** 192.168.0.100
	* **Dest MAC:** Default gateway MAC.

## Disadvantage

* Các Backend server chỉ hoạt động được chung 1 L2 network.
	* Location bị hạn chế vì các Backend phải nằm chung 1 rack hoặc datacenter với Load Balancer.
* Các hạn chế liên quan đến L3, L4 như load balancing theo port hoặc IP.
