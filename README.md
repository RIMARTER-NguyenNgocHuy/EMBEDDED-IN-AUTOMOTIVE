# EMBEDDED-IN-AUTOMOTIVE
- [Bài 1: Setup project đầu tiên trên Keil C](#bài-1-setup-project-đầu-tiên-trên-keil-c)
- [Bài 2: GPIO](#bài-2-gpio)
- [Bài 3: Don't know yet](#bài-3-dont-know-yet)
- [Bài 4: Don't know yet](#bài-4-dont-know-yet)

## Bài 1: Setup project đầu tiên trên Keil C
Quá trình: Chương trình C --> file.hex --> Vi điều khiển. Đầu tiên tạo một project trắng hoàn toàn mới trên Keil C để lập trình vi điều khiển STM32F103RCT6
1. Cấp xung clock cho ngoại vi muốn sử
  - GPIOD được kết nối với bus ABP2 nên định nghĩa và lấy giá trị địa chỉ là: địa chỉ bắt đầu của clock RCC 0x4002 1000 + address clock APB2 0x18 = 0x4002 1018.
  - Bật xung clock cho chân GPIOD tại bit thứ 5 trong thanh ghi với kĩ thuật bitmask bằng cách dịch phải 5 lần với giá trị 1
2. Cấu hình chức năng cho ngoại vi
  - Sử dụng GPIOD 2 nên dùng thanh ghi CRL 

## Bài 2: GPIO
thứ 2

## Bài 3: Don't know yet
thứ 3

## Bài 4: Don't know yet
thứ 4

