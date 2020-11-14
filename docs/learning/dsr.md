# DSR

## Load Balancing Solutions

### DR

**DR** (**D**irect **R**outing):
* **Advantages:**
	* Tốc độ truyền tải cũng như relay packet cực nhanh.
	* Backend relay trực tiếp về Client mà không cần qua Load Balancer.
		* Giải quyết được vấn đề Load Balancer bottleneck ở **NAT** solution.
* **Disadvantages:**
	* Gặp vấn đề về ARP nên cần phải cấu hình VIP trên Backend.
	* Load Balancer không quản lý với các relay packets.
		* Không phù hợp cho mô hình như QoS cho các services đứng sau Load Balancer.

### NAT

**NAT** (**N**etwork **A**ddress **T**ranslation):
* **Advantages:**
	* Tốc độ truyền tải nhanh nhưng chậm hơn **DR**.
* **Disadvantages:**
	* Do packet request và respond đều qua Load Balancer nên sẽ bị bottleneck.

## What is DSR

**DSR** viết tắt của **D**irect **S**erver **R**eturn.

**DSR** giúp **Load Balancer** routes các gói tin đến backend mà không thay đổi bất cứ gì (ngoại trừ địa chỉ MAC đích). **Backend** sẽ tiếp nhận request và trả về response trực tiếp đến client mà không qua **Load Balancer** nữa.

Trên con **Backend** phải được cấu hình 1 IP trên card loopback (được gọi là VIP - Virtual IP) tương ứng với **Load Balancer** để accept và tiếp nhận các gói tin được route từ **Load Balancer** về.

## Dataflow

![dsr-01](/docs/img/dsr-01.png)


