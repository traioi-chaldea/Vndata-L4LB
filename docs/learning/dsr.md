# DSR

## What is DSR

**DSR** viết tắt của **D**irect **S**erver **R**eturn.

**DSR** giúp **Load Balancer** routes các gói tin đến backend mà không thay đổi bất cứ gì (ngoại trừ địa chỉ MAC đích). **Backend** sẽ tiếp nhận request và trả về response trực tiếp đến client mà không qua **Load Balancer** nữa.

Trên con **Backend** phải được cấu hình 1 IP trên card loopback (được gọi là VIP - Virtual IP) tương ứng với **Load Balancer** để accept và tiếp nhận các gói tin được route từ **Load Balancer** về.

## Dataflow

![dsr-01](/docs/img/dsr-01.png)

