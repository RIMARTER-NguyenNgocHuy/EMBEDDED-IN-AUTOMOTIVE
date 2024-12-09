# EMBEDDED-IN-AUTOMOTIVE
- [Bài 1: Setup project đầu tiên trên Keil C](#bài-1-setup-project-đầu-tiên-trên-keil-c)
- [Bài 2: GPIO](#bài-2-gpio)

## Bài 1: Setup project đầu tiên trên Keil C
- Quá trình lập trình vi điều khiển: Chương trình C --> file.hex --> Vi điều khiển.
- Đầu tiên tạo một project trắng hoàn toàn mới trên Keil C để lập trình vi điều khiển STM32F103RCT6 với chức năng Blink Led
- Các bước thực hiện:
1. Cấp xung clock cho ngoại vi muốn sử.
    - GPIOD được kết nối với bus ABP2 nên định nghĩa và lấy giá trị địa chỉ là: địa chỉ bắt đầu của clock RCC 0x4002 1000 + address clock APB2 0x18 = 0x4002 1018.
    - Bật xung clock cho chân GPIOD tại bit thứ 5 trong thanh ghi với kĩ thuật bitmask bằng cách dịch phải 5 lần với giá trị 1
2. Cấu hình chế độ cho ngoại vi.
    - Sử dụng GPIO PD2 nên cấu hình CNF2, MODE2 dùng thanh ghi CRL.
    - Set 2 bit MODE2 lên 1 để chọn chế độ output,maxpeed=50 MHZ và 2 bit CNF2 lên 0 để chọn output push-pull
3. Sử dụng ngoại vi.
    - Sử dụng register ODR, output data với địa chỉ được định nghĩa là: địa chỉ GPIOD 0x40021400 + 0x0C = 0x4002140C
    - Set bit thứ 2 ODR lên 1 để tắt Led vì anode của led nối Vcc và ngược lại set bit thứ lên 0 để tắt Led. Sau đó mình sẽ viết thêm một hàm delay bằng cách sử dụng vòng lặp for chạy lệnh __asm("NOP") ước tính khoảng bao nhiêu giây để dễ theo dõi sự thay đổi trạng thái Led 
   

## Bài 2: GPIO

