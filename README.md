# undergraduate-project
#include "DSP28_Device.h"
#include "DS28_DS18B20.h"


unsigned  int  *KEY_SEL_REG= (unsigned int *)0x2E00;

unsigned char Start=0;
unsigned char End=0;
unsigned char Kuai=0;
unsigned char Man=0;
 char *msg;

 char buffer[4];

void SCI_SEND(Uint16 ex);
void SCI_MSG(char *msg);
void int2char(unsigned int number);


void main(void)
{
	float wendu=0;
	unsigned int wenduzhi=0;
    unsigned int KeyValue;

    
	InitSysCtrl();  //初始化系统函数
	

	
	DINT;
	IER = 0x0000;   //禁止CPU中断
	IFR = 0x0000;   //清除CPU中断标志
	

	
	InitPieCtrl();  //初始化PIE控制寄存器
	InitPieVectTable();  //初始化PIE中断向量表	 
	InitSpi();
    InitSci();
	
	
	
	
	InitEv();
	InitGpio();  //初始化Gpio口
	
	
   EINT;   // Enable Global interrupt INTM
   ERTM;   // Enable Global realtime interrupt DBGM
   

	KeyValue=0x0f;
    while(1)
	{   
      wendu=gettmp();
      wenduzhi= (unsigned int) wendu*10;
     int2char(wenduzhi);
     msg = "\r\n The temperature is: ";
      SCI_MSG(msg);
      SCI_MSG(buffer);
      
     
      if(wendu>35)
      {
      	EvaRegs.CMPR1=EvaRegs.CMPR1=0x3345;
      }
      if(wendu<25)
      {
      	EvaRegs.CMPR1=EvaRegs.CMPR1=0x15F9;
      }
      
      KeyValue=(*KEY_SEL_REG)&0x0f;
		KeyValue=15-KeyValue;
		if(KeyValue!=0x00)
		{
			switch(KeyValue)
			{
				case 1 :   Start=1 ; End=0      ;break;
			    case 2 :   End=1  ;  Start=0    ;break;
			    case 4 :   Kuai=1;Man=0        ;break;
			    case 8 :   Man=1; Kuai=0        ;break;
				
			}
			
		}
	  
      if(Start)
      {
      	EvaRegs.COMCONA.bit.FCOMPOE=1;    //全比较输出，PWM1-6引脚均由相应的比较逻辑驱动
      }
      if(End)
      {
      	EvaRegs.COMCONA.bit.FCOMPOE=0;    //全比较输出，PWM1-6引脚均由相应的比较逻辑驱动
      }
      if(Kuai && Start)
      {
      	EvaRegs.CMPR1=EvaRegs.CMPR1+0x0EA6;
      }
      if(Man && Start)
      {
      	EvaRegs.CMPR1=EvaRegs.CMPR1-0x0EA6;
      }
 
    }

}

void SCI_SEND(Uint16 ex)
{
	while(SciaRegs.SCICTL2.bit.TXRDY == 0);
	SciaRegs.SCITXBUF = ex;	
}

void SCI_MSG(char *msg)
{
	Uint16 i=0;
	while(msg[i] != '\0')
	{
		SCI_SEND(msg[i]);
		i++;
	}
}
void int2char(unsigned int number)
{
	buffer[0]=number/100+0x30;
    buffer[1]=(number%100)/10+0x30;
    buffer[2]=0x2E;
	buffer[3]=number%10+0x30;
	
	
}

