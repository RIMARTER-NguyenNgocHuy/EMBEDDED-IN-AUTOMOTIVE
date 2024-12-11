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
    - Sử dụng GPIO PD2 nên cấu hình CNF2, MODE2 dùng thanh ghi CRL (0..7).
    - Set 2 bit MODE2 lên 1 để chọn chế độ output,maxpeed=50 MHZ và 2 bit CNF2 lên 0 để chọn output push-pull
3. Sử dụng ngoại vi.
    - Sử dụng register ODR, output data với địa chỉ được định nghĩa là: địa chỉ GPIOD 0x40021400 + 0x0C = 0x4002140C
    - Set bit thứ 2 ODR lên 1 để tắt Led vì anode của led nối Vcc và ngược lại set bit thứ lên 0 để tắt Led. Sau đó mình sẽ viết thêm một hàm delay bằng cách sử dụng vòng lặp for chạy lệnh __asm("NOP") ước tính        khoảng bao nhiêu giây để dễ theo dõi sự thay đổi trạng thái Led
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
- Dùng thư viện SPL để điều khiển ngoại vi GPIO thực hiện các chức năng như: Blink Led, kết hợp Nút nhấn để thay đổi trạng thái Led
- Tổng quan các bước:
  -- Bật xung clock cho ngoại vi muốn sử dụng --> Cấu hình chân --> Sủ dụng chức năng của ngoại vi




