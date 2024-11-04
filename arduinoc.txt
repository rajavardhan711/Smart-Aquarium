#include <Wire.h>
#include <Adafruit_INA219.h>
#include <LiquidCrystal.h>
const int rs = 8, en = 9, d4 = 10, d5 = 11, d6 = 12, d7 = 13;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
Adafruit_INA219 ina219;
#include "DHT.h“
float R1 = 30000.0;
float R2 = 7500.0;

// Float for Reference Voltage
float ref_voltage = 5.0;
int cnt=0;
#define DHTPIN 7
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

void setup(void)
{
	Serial.begin(9600);
	lcd.begin(16,2);
	lcd.print(" WELCOME ");
	dht.begin();
	uint32_t currentFrequency;
	if (!ina219.begin()) {
		Serial.println("Failed to find INA219 chip");
	while (1) { delay(10); }
	}
}

void loop(void)
{
	float shuntvoltage = 0;
	float busvoltage = 0;
	float current_mA = 0;
	float loadvoltage = 0;
	float power_mW = 0;
	int cs=0;
	shuntvoltage = ina219.getShuntVoltage_mV();
	busvoltage = ina219.getBusVoltage_V();
	current_mA = ina219.getCurrent_mA();
	power_mW = ina219.getPower_mW();
	loadvoltage = busvoltage + (shuntvoltage / 1000);
	if(busvoltage<1.1)
		busvoltage=0;
	if(current_mA<1.5)
		current_mA=0;
	if(power_mW<1.5)
		current_mA=0;
	int bp=map(busvoltage,2.5, 3.9,1,100);
	int t = dht.readTemperature();
	float b1v=((analogRead(A0)*5)/1024.0)/(R2/(R1+R2));
	float b2v=((analogRead(A1)*5)/1024.0)/(R2/(R1+R2))-b1v;
	float b3v=((analogRead(A2)*5)/1024.0)/(R2/(R1+R2))-b2v-b1v;
	//busvoltage=21.4;
	//current_mA=10.7;
	//t=32.6;
	//b1v=7.1;
	//b2v=7.2;
	//b3v=6.2;
	cnt=cnt+1;
	if(cnt>15)
	{
		cnt=0; Serial.print(busvoltage);
		Serial.print(',');
		Serial.print(current_mA);
		Serial.print(',');Serial.print(t);
		Serial.print(',');
		Serial.print(b1v);
		Serial.print(',');
		Serial.print(b2v);
		Serial.print(',');
		Serial.print(b3v);
		Serial.print(",0\n");
	}
	lcd.clear();
	lcd.print("V:"+String(busvoltage,1) + " I:"+String(current_mA));
	lcd.setCursor(0,1);
	lcd.print(String(b1v,1)+ " "+String(b2v,1)+ " "+String(b3v,1)+" T:“+String(t));
	delay(1000);
}