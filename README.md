# INTERFACING-TEMPERATURE-SENSOR-In-Embedded-C
#include <LPC17xx.H>
#include "GLCD.H"
#include "Serial.h"
#define __FI 1
unsigned int Adc;
unsigned long Low_adc,High_adc,relay;
read_adc()
{
unsigned long status,i;
LPC_GPIO0->FIOPIN |= (1 << 26);
status = ((LPC_GPIO1->FIOPIN & 0x00080000) >> 19);
while((status & 0x01) != 0x01)
status = ((LPC_GPIO1->FIOPIN & 0x00080000) >> 19);
LPC_GPIO0->FIOPIN &= ~(1 << 26);
LPC_GPIO0->FIOPIN &= ~(1 << 25);
LPC_GPIO0->FIOPIN &= ~(1 << 23);
for(i=0;i<10000;i++);
Low_adc = (LPC_GPIO0->FIOPIN & 0x007F8000);
Low_adc = (Low_adc >> 15) & 0xFF;
LPC_GPIO0->FIOPIN |= (1 << 23);
LPC_GPIO0->FIOPIN &= ~(1 << 24);
for(i=0;i<10000;i++);
High_adc = (LPC_GPIO0->FIOPIN & 0x00078000);
High_adc = (High_adc >> 15) & 0x0F;
LPC_GPIO0->FIOPIN |= (1 << 24);
LPC_GPIO0->FIOPIN |= (1 << 25);
LPC_GPIO0->FIOPIN &= ~(1 << 26);
}
main()
{
float Temp,Vol,Res;
unsigned char Temp1,Temp2,Temp3;
LPC_SC->PCONP |= (1<<15);
LPC_GPIO0->FIODIR |= 0x7F800000;
LPC_GPIO1->FIODIR |= (1 << 20);
LPC_PINCON->PINMODE0 |= 0xC0000000;
LPC_PINCON->PINMODE1 |= 0x00003FFF;
LPC_GPIO1->FIOPIN &= ~(1 << 20);
LPC_GPIO0->FIOPIN &= ~(1 << 26);
relay = 10;
SER_Init();
#ifdef __USE_LCD
GLCD_Init();
GLCD_Clear(White);
GLCD_SetBackColor(Blue);
GLCD_SetTextColor(White);
GLCD_DisplayString(0, 0, __FI, " ECE ");
GLCD_DisplayString(1, 0, __FI, " KRCE ");
GLCD_DisplayString(2, 0, __FI, " EMBEDDEDLAB ");
GLCD_SetBackColor(White);
GLCD_SetTextColor(Blue);
GLCD_DisplayString(3, 0, __FI, "TEMP TRANSDUCER TXDR");
#endif
while(1)
{
read_adc();
Adc = High_adc;
Adc <<= 8;
Adc = Adc | Low_adc;
if( (Adc > 0x0656) && (relay != 0))
{
GLCD_DisplayString(5, 0, __FI, "Relay OFF");
SER_SendString("\n\rRELAY OFF");
LPC_GPIO1->FIOPIN &= ~(1 << 20);
relay = 0;
}
else if ( (Adc < 0x5B9) && (relay!= 1))
{
SER_SendString("\n\rRELAY ON");
GLCD_DisplayString(5, 0, __FI, "Relay ON ");
LPC_GPIO1->FIOPIN |= (1 << 20);
relay = 1;
}
Vol =-((Adc/10)*0.000488);
Res =((100*(1.8-Vol)-100*Vol)*100) /(100*Vol + 100*(1.8+Vol));
Res = Res - 100;
Temp = Res/ 0.384;
Temp1 = Temp;
Temp2 = (0x30 + (Temp1 / 0x0A));
Temp3 = (0x30 + (Temp1 % 0x0A));
GLCD_DisplayString(6, 0, __FI, "Temperature=");
GLCD_DisplayChar(6, 13, __FI, Temp2);
GLCD_DisplayChar(6, 14, __FI, Temp3);
SER_SendString(" Temperature = ");
SER_PutChar(Temp2);
SER_PutChar(Temp3);
GLCD_DisplayString(6, 16, __FI, "'C");
SER_SendString("'C\n\r");
}
}
