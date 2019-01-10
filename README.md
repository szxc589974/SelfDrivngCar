自走車期末戰程式碼
-------------------------------
#include <LWiFi.h>
#define R_F 12
#define R_B 11
#define L_F 13
#define L_B 14

#define U_F 0
#define U_L 1
#define U_R 2
#define SSID "CSIE-WLAN-Sparq"
#define PASSWD "wificsie"

#define TCP_IP "192.168.209.33"
#define TCP_PORT 5000

#define G 15
#define B 16
#define R 17

WiFiClient wifiClient;
static char buf[32];
static int messageLen;
int aF[2],aL[2],aR[2];

int TrigPins[3] = {2, 4, 6};  // F, L, R
int EchoPins[3] = {3, 5, 7};  // F, L, R
int car_speed =135;


  int AtoD[6] = {2,1,1,1,2,2};
  int BtoA[8] = {1,2,1,0,0,0,1};
  int CtoB[10] = {0,2,1,0,0,2,1,1,0,2};
  int DtoC[8] = {1,1,2,0,0,0,0,1}; 
  int current=0;

int GetDistance(int trig, int echo)
{
  digitalWrite(trig, LOW);
  delayMicroseconds(2);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);

  int duration;
  duration = pulseIn(echo, HIGH);
  int distance = duration / 29 / 2;
  return distance;
}
void setup(){
  Serial.begin(9600);
  pinMode(TrigPins[U_F], OUTPUT);
  pinMode(TrigPins[U_L], OUTPUT);
  pinMode(TrigPins[U_R], OUTPUT);
  pinMode(EchoPins[U_F], INPUT);
  pinMode(EchoPins[U_L], INPUT);
  pinMode(EchoPins[U_R], INPUT);
  pinMode(R_F,OUTPUT);
  pinMode(R_B,OUTPUT);
  pinMode(L_F,OUTPUT);
  pinMode(L_B,OUTPUT);
  
  pinMode(G,OUTPUT);///////////
  pinMode(B,OUTPUT);
  pinMode(R,OUTPUT);
  int status = WL_IDLE_STATUS;
  while (status != WL_CONNECTED)
  {
    status = WiFi.begin(SSID,PASSWD);
  }

  Serial.begin(9600);
  Serial.println("WiFi Connect");
  wifiClient.connect(TCP_IP,TCP_PORT);
  Serial.println("Server Connect");
  delay(1000);
  wifiClient.write("join Su B");  //join <ID> <team>
}
void forward () {
  analogWrite(L_F, car_speed);
  analogWrite(L_B, 0);
  analogWrite(R_F, car_speed);
  analogWrite(R_B, 0);
}
void back(){
  analogWrite(R_B,car_speed);
  analogWrite(R_F,LOW);
  analogWrite(L_B,car_speed);
  analogWrite(L_F,LOW);
}

void STOP(){
  analogWrite(R_F,0);
  analogWrite(R_B,0);
  analogWrite(L_F,0);
  analogWrite(L_B,0);
}
void turnleft(){
  analogWrite(R_F,85);
  analogWrite(R_B,0);
  analogWrite(L_F,0);
  analogWrite(L_B,85);
}
void turnright(){
  analogWrite(R_B,85);
  analogWrite(R_F,0);
  analogWrite(L_F,85);
  analogWrite(L_B,0);
}

void loop(){
  analogWrite(R,255);
  analogWrite(B,50);
  analogWrite(G,10);
  int j = 0;
   while (wifiClient.available()) {
       buf[j++] = wifiClient.read();
       delayMicroseconds(10);
   }
   if (j != 0) {
     buf[j] = '\0';
     Serial.println(buf);
   }
   j = 0; 
  int dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  int dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  int dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  
  while(getPos() != 56) delay(1000);
  digitalWrite(R, HIGH);
  Bto();
  wait();
  digitalWrite(R, LOW);
  while(getPos() != 63) delay(1000);
  digitalWrite(R, HIGH);
  Ato();
  wait();
  digitalWrite(R, LOW);
  while(getPos() != 7) delay(1000);
  digitalWrite(R, HIGH);
  Dto();
  wait();
  digitalWrite(R, LOW);
  while(getPos() != 0) delay(1000);
  digitalWrite(R, HIGH);
  Cto();
  digitalWrite(R, LOW);
   
//  Ato();
//  wait();
//  Dto();
//  wait();
//  Cto();
//  wait();
//  Bto();
//  wait();
//  Ato();
}
int Ato()
{ 

  int Ltime = 650;
  int Rtime = 600;
  int dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  int dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  int dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  int current = 0;
  while(current<6){
    if((dL > 40) && (dR < 20))
    {
      if(AtoD[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(380);
        turnleft();
        delay(Ltime);
        forward();
        delay(600);
        cur();
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
      }
      else if(AtoD[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(150); 
       cur();
       test();
      dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
      dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
      dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
      cur();
      test();
    }
    }
    else if((dR > 40) && (dL<20))
    {
      if(AtoD[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        delay(380);
        turnright();
        delay(Rtime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
      else if(AtoD[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(300); 
       cur();
       test();
      }
      dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
      dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
      dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    else if((dR > 40) && (dL > 40)){
        if(AtoD[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(380);
        turnleft();
        delay(Ltime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        if(AtoD[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        cur();
        delay(380);
        turnright();
        delay(Rtime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        else if(AtoD[current]==0){
         current+=1;
         Serial.println("我沒轉");
         forward();
         delay(250); 
         cur();
         test();
        }
    }
    else{
      forward();
      Serial.println("沒事沒事");
      delay(120);
      cur();
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  }
  forward();
  delay(2500);
}//AtoD


int Bto()
{ 
  int Rtime = 650;
  int Ltime = 680;
  STOP();
  delay(50);
  int dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  int dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  int dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  int current = 0;
  while(current<8){
    if((dL > 30) && (dR < 20))
    {
      if(BtoA[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(380);
        turnleft();
        delay(Rtime);
        forward();
        delay(150);
        cur();
      }
      else if(BtoA[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(500); 
       cur();
       test();
      
      dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
      dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
      dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
      cur();
      test();
    }
    }
    else if((dR > 30) && (dL<20))
    {
      if(BtoA[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        delay(345);
        turnright();
        delay(Rtime);
        cur();
        forward();
        delay(600);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
      else if(BtoA[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(300); 
       cur();
       test();
      }
    }
    else if((dR > 40) && (dL > 40)){
        if(BtoA[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(380);
        turnleft();
        delay(Ltime);
        forward();
        delay(300);
        STOP();
        delay(50);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        if(BtoA[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        delay(380);
        turnright();
        delay(Rtime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        else if(BtoA[current]==0){
         current+=1;
         Serial.println("我沒轉");
         forward();
         delay(250); 
         cur();
         test();
        }
    }
    else{
      forward();
      Serial.println("沒事沒事");
      delay(120);
      cur();
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  }
  forward();
  delay(2500);
}//BtoA

int Cto()
{ 
  int Ltime = 675;
  int Rtime = 685;
  int dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  int dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  int dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  int current = 0;
  while(current<10){
    if((dL > 40) && (dR < 20))
    {
      if(CtoB[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(450);
        turnleft();
        delay(Ltime);
        forward();
        delay(200);
        cur();
      }
      else if(CtoB[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(500); 
//       cur();
       test();
     }
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    else if((dR > 40) && (dL<20))
    {
      if(CtoB[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        delay(500);
        turnright();
        delay(640);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
      else if(CtoB[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(500); 
       cur();
       test();
      }
    }
    else if((dR > 30) && (dL > 30)){
        if(CtoB[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(400);
        turnleft();
        delay(Ltime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        if(CtoB[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        cur();
        delay(380);
        turnright();
        delay(Rtime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        else if(CtoB[current]==0){
         current+=1;
         Serial.println("我沒轉");
         forward();
         delay(120); 
         cur();
         test();
        }
    }
    else{
      forward();
      Serial.println("沒事沒事");
      delay(120);
      cur();
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  }
  forward();
  delay(500);
  STOP();
  delay(200);
}//CtoB

int Dto()
{ 
  forward();
  delay(800);
  int Ltime = 680;
  int Rtime = 640;
  int dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  int dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  int dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  int current = 0;
  while(current<8){
    if((dL > 40) && (dR < 20))
    {
      if(DtoC[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(390);
        turnleft();
        delay(Ltime);
        forward();
        delay(200);
        cur();
      }
      else if(DtoC[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(350); 
       cur();
       test();
    }
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    else if((dR > 40) && (dL<20))
    {
      if(DtoC[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        delay(380);
        turnright();
        delay(Rtime);
        forward();
        delay(300);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
      else if(DtoC[current]==0){
       current+=1;
       Serial.println("我沒轉");
       forward();
       delay(350); 
       cur();
       test();
      }
      dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
      dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
      dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    else if((dR > 40) && (dL > 40)){
        if(DtoC[current]==2){
        current+=1;
        forward();
        Serial.println("左轉");
        delay(370);
        turnleft();
        delay(Ltime);
        forward();
        delay(120);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        if(DtoC[current]==1){
        current+=1;
        forward();
        Serial.println("右轉");
        delay(370);
        turnright();
        delay(Rtime);
        forward();
        delay(120);
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
        cur();
        test();
      }
        else if(DtoC[current]==0){
         current+=1;
         Serial.println("我沒轉");
         forward();
         delay(350); 
         cur();
         test();
        }
        dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
        dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
        dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    else{
      forward();
      Serial.println("沒事沒事");
      delay(120);
      cur();
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
    }
    dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
    dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
    dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  }
  forward();
  delay(2500);
}//DtoC

int wait()
{
  int dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  if(dF<5)
  {
    back();
    delay(250);
    cur();
    turnright();
    delay(1100);
    cur();
    back();
    delay(700);
    STOP();
    delay(150);
  }
  else if(dF<8)
  {
    back();
    delay(150);
    cur();
    turnright();
    delay(1100);
    cur();
    back();
    delay(500);
    cur();
    STOP();
    delay(150);
  }
  else 
  {
    cur();
    turnright();
    delay(1100);
    cur();
    back();
    delay(500);
    cur();
    STOP();
    delay(150);
  }
}

int cur()
{
  int dF, dL, dR;
  dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  if (dL < 7 || dL > 280)
  {
    back();
    delay(70);
    turnright();
    delay(55);
  }
  if (dR < 7 || dR > 280)
  {
    back();
    delay(60);
    turnleft();
    delay(55);
  }
  if ((dR<25) && (dR>20))
  {
    back();
    delay(70);
    turnright();
    delay(50);
  }
  if ((dL<30) && (dL>15))
  {
    back();
    delay(70);
    turnleft();
    delay(75);
  }
}

int test()
{
  int dF, dL, dR;
  int i, diffR, diffL, diffF;
  int moveL, moveR, moveF;
  dF = GetDistance(TrigPins[U_F], EchoPins[U_F]);
  dL = GetDistance(TrigPins[U_L], EchoPins[U_L]);
  dR = GetDistance(TrigPins[U_R], EchoPins[U_R]);
  forward();
  aF[0] = dF;
  diffF = 5;

  if (dF > 250) {
    dF = 1;
  }
  if (aF[0] > 250) {
    dF = 1; 
  }
  if (aF[1] > 250){
    dF = 1;
  }
  diffF = abs(aF[1] - aF[0]);
   Serial.print("diff: ");
   Serial.print(diffF);
  if (diffF < 1)
  {
    moveF = 0;
  }
  else
  {
    moveF = 1;
  }
//forward

  aL[0] = dL;
  diffL = 5;

  if (dL > 250) {
    dL = 1;
  }
  if (aL[0] > 250) {
    dL = 1; 
  }
  if (aL[1] > 250){
    dL = 1;
  }
  diffL = abs(aL[1] - aL[0]);
   Serial.print("diff: ");
   Serial.print(diffL);
  if (diffL < 1)
  {
    moveL = 0;
  }
  else
  {
    moveL = 1;
  }
//left

  aR[0] = dR;
  diffR = 5;

  if (dR > 250) {
    dR = 1;
  }
  if (aR[0] > 250) {
    dR = 1; 
  }
  if (aR[1] > 250){
    dR = 1;
  }
  diffR = abs(aR[1] - aR[0]);
   Serial.print("diff: ");
   Serial.print(diffR);
  if (diffR < 1)
  {
    moveR = 0;
  }
  else
  {
    moveR = 1;
  }
  if ((moveF == 0)&&(moveR == 0)&&(moveL == 0))
  {
    Serial.println("偵測卡住!!!!!!!!");
    STOP();
    delay(200);
    back();
    delay(550);
    if (dL<dR)
  {
      Serial.println("卡住後右轉");
      turnright();
      delay(500);
  }
    else
  {
      Serial.println("卡住後左轉");
      turnleft();
      delay(250);
  }
  }
}

void getRequest(){
  int i;
  for(i = 0; i < 32; i++)
    buf[i] = '\0';

  i = 0;
  while(wifiClient.available()){
    buf[i++] = wifiClient.read();
    delayMicroseconds(10);
  }

  if(i != 0){
    Serial.println(buf);
  }
}

int getPos(){
  int x, y;
  do{
    x = y = -3;
    wifiClient.write("position");
    getRequest();

    int i = 0;
    while(buf[i] != '\0'){
      if(buf[i++] == ' ')
        if(x == -3)
          x = buf[i] - '0';
        else
          y = buf[i] - '0';
    }

    if(y == -3)
      delay(1000);
  }while(y == -3);

  return x - 8*y + 56;
}
