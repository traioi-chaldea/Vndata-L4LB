# Layer 4 Load Balancing

## Reality Projects

| Company | Project | Technology | OSI Layer | OpenSource? |
| --- | --- | --- | --- | --- |
| Cloudflare | [Unimog](https://blog.cloudflare.com/unimog-cloudflares-edge-load-balancer/) | XDP | L4/L7 | |
| Github | [GLB](https://github.blog/2018-08-08-glb-director-open-source-load-balancer/) | DPDK/XDP | L4 | x |
| Google | [Maglev](https://research.google.com/pubs/archive/44824.pdf) | DPDK | L4/L7 | |
| Facebook | [Katran](https://engineering.fb.com/2018/05/22/open-source/open-sourcing-katran-a-scalable-network-load-balancer/) | XDP | L4/L7 | x |
| HAproxy | [HAproxy](https://github.com/haproxy/haproxy) | DPDK | L4/L7 | x |

## Disadvantages of L3DSR

* Nếu phía sau Load Balancer có nhiều hơn 2 services (không phải Backend Server) đang chạy thì phải cần số lượng IP tương ứng.
	* Do Layer 3 không handle được field Port header như Layer 4 (TCP header) nên phải chia Sevices theo IP.

## L4DSR Technologies

### IP-in-IP Encapsulation

**L4LB** sử dụng 1 IP để forward packets cho nhiều Backend ở phía sau Load Balancer. Tuy nhiên các packets này cần đánh dấu (mark) để phân biệt rằng packet này phù hợp cho Services nào. **L4LB** sẽ sử dụng 5 fields quan trọng được trích ra từ các Layer Header tương ứng để đóng gói và xác định rằng packets này có flow nào phù hợp đi về Backend nào.

	* **Source Address (Layer 3):** Lưu lại Client IP để bypass checksum và sử dụng cho Backend DR về Client.
	* **Destination Address (Layer 3):** Xác định Load Balancer/NIC tương ứng với IP.
	* **Source Port (Layer 4):** Lưu lại Client Port để không bị break đường socket Client-Server.
	* **Destination Port (Layer 4):** Xác định Service mà Client truy cập.
	* **HTTP Request (Layer 7):** Xác định các Header Request  mà Client truy cập.

**IP-in-IP encapsulation** giúp lưu giữ các thông tin đã đóng gói của packets khi Load Balancer forward packets về các Backend.

Tương tự như L3DSR, VIP phải được config trên card loopback để so sánh với thông tin đóng gói của packets. Phần này giúp response packets được trả về trực tiếp cho Client thay vì Load Balancer.
