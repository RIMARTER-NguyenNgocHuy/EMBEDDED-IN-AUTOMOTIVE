# EMBEDDED-IN-AUTOMOTIVE
- [Bài 1: Setup project đầu tiên trên Keil C](#bài-1-setup-project-đầu-tiên-trên-keil-c)
- [Bài 2: GPIO](#bài-2-gpio)
- [Bài 3: Interrupt và Timer](#bài-3-interrupt-và-timer)


## Bài 1: Setup project đầu tiên trên Keil C
- Quá trình lập trình vi điều khiển: Chương trình C --> file.hex --> Vi điều khiển.
- Đầu tiên tạo một project trắng hoàn toàn mới trên Keil C để lập trình vi điều khiển STM32F103RCT6 với chức năng Blink Led.
- Các bước thực hiện:
1. Cấp xung clock cho ngoại vi muốn sử.
    - GPIOD được kết nối với bus ABP2 nên định nghĩa và lấy giá trị địa chỉ là: địa chỉ bắt đầu của clock RCC 0x4002 1000 + address clock APB2 0x18 = 0x4002 1018.
    - Bật xung clock cho chân GPIOD tại bit thứ 5 trong thanh ghi với kĩ thuật bitmask bằng cách dịch phải 5 lần với giá trị 1.
2. Cấu hình chế độ cho ngoại vi.
    - Sử dụng GPIO PD2 nên cấu hình CNF2, MODE2 dùng thanh ghi CRL (0..7).
    - Set 2 bit MODE2 lên 1 để chọn chế độ output,maxpeed=50 MHZ và 2 bit CNF2 lên 0 để chọn output push-pull.
3. Sử dụng ngoại vi.
    - Sử dụng register ODR, output data với địa chỉ được định nghĩa là: địa chỉ GPIOD 0x40021400 + 0x0C = 0x4002140C.
    - Set bit thứ 2 ODR lên 1 để tắt Led vì anode của led nối Vcc và ngược lại set bit thứ lên 0 để tắt Led. Sau đó mình sẽ viết thêm một hàm delay bằng cách sử dụng vòng lặp for chạy lệnh __asm("NOP") ước tính        khoảng bao nhiêu giây để dễ theo dõi sự thay đổi trạng thái Led.
- Code:
    ```
    #include "stm32f10x.h" 

    typedef struct
    {
    	unsigned int CRL;
    	unsigned int CRH;
    	unsigned int IDR;
    	unsigned int ODR;
    	unsigned int BSRR;
    	unsigned int BRR;
    	unsigned int LCKR;
    } GPIO_typeDef;
    
    typedef struct
    {
    	unsigned int CR;
    	unsigned int CFGR;
    	unsigned int CIR;
    	unsigned int APB2RSTR;
    	unsigned int APB1RSTR;
    	unsigned int AHBENR;
    	unsigned int APB2ENR;
    	unsigned int APB1ENR;
    	unsigned int BDCR;
    	unsigned int CSR;
    } RCC_typeDef;
    
    #define RCC_APB2ENR ((RCC_typeDef *) 0x40021000)
    #define GPIOD ((GPIO_typeDef *) 0x40011400)
    #define GPIOC ((GPIO_typeDef *) 0x40011000)
    
    void RCCconfig();
    void GPIOconfig();
    void delay(unsigned int  timedelay);
    void WritePin(GPIO_TypeDef *GPIO_Port, unsigned int Pin, unsigned int state);
    
    unsigned int j = 0, k = 0;
    
    int main(){
    	RCCconfig();
    	GPIOconfig();
    	while(1){
            if((GPIOC->IDR & (1 << 13))  == 0){
    			WritePin(GPIOD, 2, 0);
    			delay(10000000);
    		}
    		else 
    			WritePin(GPIOD, 2, 1);
    			delay(10000000);
    		
    //		WritePin(GPIOD, 2, 0);
    //		delay(10000000);
    //		WritePin(GPIOD, 2, 1);
    //		delay(10000000);
    	}
    }

    void RCCconfig(){
    	RCC->APB2ENR |= (1 << 5) | (1 << 4);
    }

    void GPIOconfig(void){
    		GPIOD->CRL |= (3 << 8);  
    		GPIOD->CRL &= ~(3 << 10); 
    	
    		GPIOC->CRH |= (8 << 22);
    		GPIOC->CRH &= ~(7 << 20);
    		GPIOC->ODR |= (1 << 13);
    		//GPIOD->CRL &= ~(0xF << 20); 
    		//GPIOC->CRH |= (0x04 << 20); 
    }

    void WritePin(GPIO_TypeDef *GPIO_Port, unsigned int Pin, unsigned int state){
    	if(state == 1) GPIO_Port->ODR |= (1 << Pin);
    	else GPIO_Port->ODR &= ~(1 << Pin);
    }

    void delay(unsigned int timedelay) {
        for (unsigned int i = 0; i < timedelay; i++) {
            __asm("NOP");
        }
        j++;
    }
    ```

## Bài 2: GPIO
- Dùng thư viện SPL để điều khiển ngoại vi GPIO trên MCU STM32F412RET6 thực hiện các chức năng như: Blink Led, kết hợp Nút nhấn để thay đổi trạng thái Led.
- Tổng quan các bước: Bật xung clock cho ngoại vi muốn sử dụng --> Cấu hình chân --> Sủ dụng chức năng của ngoại vi.
1. Bật xung clock cho ngoại vi.
   - Sử dụng chân GPIO B2 để làm Led có sẵn trên board F4, nên kích hoạt xung clock ở đường bus AHB1 vì GPIOB được nối với đường bus đó.
   - Tương tự cho Nút nhấn ở chân GPIO C13.
2. Cấu hình chân
   - Sử dụng kiểu dữ liệu GPIO_InitTypeDef cho cấu trúc biến GPIO_InitStruct để truy cập vào các trường.
   - Chọn Mode cho GPIOB2 (Led) là output, kiểu putpull, tốc độ tối đa. Sau đó dùng hàm GPIO_init để ghi giá trị đã cấu hình vào thanh ghi.
   - Chọn Mode cho GPIOC13 (Nút nhấn) là input, kiểu điện trở kéo xuống PuPd_DOWN và dùng hàm GPIO_init để ghi giá trị vào thanh ghi
3. Lập trình chức năng cho ngoại vi.
   - Sử dụng các hàm đọc trạng thái nút nhấn GPIO_ReadInputDataBit để theo dõi và set sự thay đổi trạng thái của led theo những lần nút nhấn được nhấn
   - Theo dõi trạng thái led bằng hàm GPIO_ReadOutputDataBit để thay đổi trạng thái hiện tại của led khi có sự kiện nhấn nút
   - Delay một chút thời gian để chương trình không chạy quá nhanh, tránh trường hợp lỗi gì đó không mong muốn
  
Code:
```
#include "stm32f4xx.h"
//#include "stm32f4xx_rcc.h"
//#include "stm32f4xx_gpio.h"

void RCCconfig();
void GPIOconfig();
void delay(uint32_t timedelay);

int main() {
    RCCconfig();
    GPIOconfig();
    while (1) {
//			GPIO_ToggleBits(GPIOB, GPIO_Pin_2);
			if(GPIO_ReadInputDataBit(GPIOC, GPIO_Pin_13) == 0){
				while(GPIO_ReadInputDataBit(GPIOC, GPIO_Pin_13) == 0);
				if(GPIO_ReadOutputDataBit(GPIOB, GPIO_Pin_2)){
					GPIO_ResetBits(GPIOB, GPIO_Pin_2);
				} else {
					GPIO_SetBits(GPIOB, GPIO_Pin_2);
				}
			}
			delay(50000);
    }
}

void RCCconfig() {
//    RCC->AHB1ENR |= 0x000001F8;
	RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOB | RCC_AHB1Periph_GPIOC, ENABLE);
}

void GPIOconfig() {
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_2;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_OUT;
	GPIO_InitStruct.GPIO_OType = GPIO_OType_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IN;
	GPIO_InitStruct.GPIO_PuPd = GPIO_PuPd_DOWN;
	GPIO_Init(GPIOC, &GPIO_InitStruct);
}

void delay(uint32_t timedelay) {
    uint32_t i;
    for (i = 0; i < timedelay; i++) {
        __asm("NOP");
    }
}
```

## Bài 3: Interrupt và Timer
- Interrupt là một cơ chế sự kiện có ngắt ben trong vi điều khiển hoặc ngắt ngoài. Interrupt yêu cầu MCU thực hiện mot sự kiện xảy ra trong chương trình ngắt tại thời điểm nào đó đang chạy chương trình chính. Có nghĩa là khi ngắt xảy ra chương trình chính sẽ ngừng hoạt động và nhảy vào hàm ngắt để thực thi chương trình bên trong đó. Mỗi sự kiện ngắt sẽ có hàm phục vụ cho việc ngắt (ISR)
	- Trong MCU ta có thanh ghi Program Counter (PC) sẽ chỉ đến địa chỉ thực thi tiếp theo của chương trình và lưu địa chỉ nhớ của lệnh tiếp theo cần thực thi. Đóng vai trò như một cái gì đó chỉ đường từ chương trình chính chuyển đến chương trình ISR và quay lại tiếp tục ở chương trình chính. Nói rõ hơn là trước khi có ngắt xảy ra PC chứa địa chỉ của lệnh tiếp theo mà chương trình sẽ thực thi. Nếu có ngắt xảy ra CPU sẽ tạm dừng chạy, lúc này thì PC sẽ lưu lại địa chỉ của lệnh đang thực thi, để lúc chạy ngắt xong có thể quay lại tại địa chỉ mà nó đã lưu ở trước đó. Và các địa chỉ trong bộ nhớ của ISR được xác định bởi vecto ngắt. Sau khi xử lý ngắt xong PC sẽ khôi phục từ stack là nơi địa chỉ của lệnh tiếp theo đã được lưu lại trước khi có ngắt. Để chương trình có thể tiếp tục chạy từ nơi nó tạm dừng để nhảy vào ISR.
	- Một số ngắt hay sử dụng như là ngắt ngoài, ngắt timer và ngắt truyền thông:
 		- Ngắt ngoài: khi có một sự kiện ngắt ở ngoài tác động đến chân GPIO của MCU như nút nhấn, nhận biết có ngắt là khi có sự biến đổi điện áp nào đó từ mức cao xuống mức thấp hoặc ngược lại.
   		- Ngắt timer: sử dụng bộ đếm Timer để nhận biết khi giá trị timer đếm tới khi nó bị tràn và reset lại giá trị khi bị tràn.
     		- Ngắt truyền thông: được sử dụng và xảy ra khi có sự truyền nhận tín hiệu của MCU với MCU khác hoặc với thiết bị khác qua các giao thức có trong MCU. Ví dụ như ta có 2 MCU cần gửi, nhận data với 		nhau. Lúc này ví dụ như MCU1 có chương trình 1 thực thi trong 2s - chương trình 2 thực thi trong 1s và MCU2 chương trình 1 là 1s - chương trình 2 1s, thì sẽ có sự chêch lệnh thời gian thực thi 		của 2 MCU dẫn đến MCU1 đã chạy đến hàm gửi data nhưng MCU2 đã chạy qua hàm nhận data vì sự chêch lệnh thời gian. Vì vậy cần có ngắt để khi có data từ MCU1 gửi đến MCU2 qua một giao thức nào đó 		thì chương trình của MCU2 sẽ tạm dừng hoạt động và nhảy đến hàm ngắt để gọi hàm nhận data. Khi thực hiện xong ngắt thì tiếp tục với chương trình chính của nó nhờ sự dẫn đường của thanh ghi PC.
- Timer 


   




