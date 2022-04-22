#include<LiquidCrystal.h>
#include<TinyGPS.h>

#include<SoftwareSerial.h>
const byte tx=10,rx=11;
SoftwareSerial esp8266(rx,tx);
String ssid = "meghanadoddmane" ;
String password = "tentimesone" ;
String api = "EVCC2N8S38IBHL42" ; 
String host = "api.thingspeak.com" ;
String port = "80" ;

const int temp_pin=A0;
const int buzz=9;
const int button=8;
const int rs = 7, en = 6, d4 = 5, d5 = 4, d6 = 3, d7 = 2;

LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

float temp;
float lat=11,lon=22;

TinyGPS gps;
void setup()
{
  pinMode(buzz,OUTPUT);
  pinMode(button,INPUT_PULLUP);
  lcd.begin(16,2);
  Serial.begin(9600);
  esp8266.begin(115200);
  pinMode(temp_pin,INPUT);
  connectwifi();
}


  void loop()
  {   
   senddata();
   delay(5000);
   int a=digitalRead(button);
   if(a==0)
   {
   digitalWrite(buzz,HIGH);
   lcd_str("EMERGENCY!!!!",0,0);
   lcd_str("HELP......",0,1);  
   temp=analogRead(temp_pin)*0.00488*100;
   Serial.print("TEMPERATURE:");
   Serial.println(temp,0);
   delay(1000);
   }
   else if(a==1)
   {
      digitalWrite(buzz,LOW);
   }
   lcd.clear();
    gps_print();
  }

 
void lcd_str(String str,char col,char row)
{
 lcd.setCursor(col,row) ;
 lcd.print(str);
}


void gps_print()
{
  while(Serial.available())
  {
    if(gps.encode(Serial.read()))
    {
     gps.f_get_position(&lat,&lon);
     Serial.print("latitude");
     Serial.println(lat,4);
     Serial.print("longitude");
     Serial.println(lon,4);
     delay(2000);
    }
  }
}




void sendcommand(String command, int maxtime, char readreply[]) 
{

boolean found = false;
  
  Serial.print(". at command => ");
  Serial.print(command);
  Serial.print(" ");
  esp8266.println(command);
         while (maxtime--)
         {
            if (esp8266.find(readreply)) 
             {
                 found = true; break;
             }
        }

  if (found)
  {
    Serial.println("OK Done");
  }
 else
  {
    Serial.println("Fail");
  }

}

void connectwifi()
{
sendcommand("AT",5,"OK");
sendcommand("AT+CWMODE=1",5,"OK");
sendcommand("AT+CWJAP=\"" + ssid + "\",\""+ password + "\"" , 20 ,"OK");
}

void senddata()
{
  String getData1 = "GET /update?api_key="+ api ;
  String getData2 = "&field1="+String(temp);
  sendcommand("AT+CIPMUX=0",5,"OK");
     sendcommand("AT+CIPSTART=\"TCP\",\""+ host + "\"," + port , 15, "OK");
 sendcommand("AT+CIPSEND=" + String(getData1.length() + getData2.length()+ 2), 7, ">");

  delay(500);

  esp8266.println(getData1+getData2);
  Serial.println(getData1 + getData2);
  delay(3500);

  sendcommand("AT+CIPCLOSE=0", 5, "OK");  
  Serial.println("---------------------------");
}
