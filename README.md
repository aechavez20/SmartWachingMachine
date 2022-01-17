# SmartWachingMachine

//Smart Waching Maching 
//Grupo#1
//Laboratorio de sistemas embebidos
//Integrantes: Carlos Cordero y Alex Chavez

//**Motor Lavado**
int IN1 = 8;      // pin digital 8 de Arduino a IN1 de modulo controlador
int IN2 = 9;      // pin digital 9 de Arduino a IN2 de modulo controlador
int IN3 = 10;     // pin digital 10 de Arduino a IN3 de modulo controlador
int IN4 = 11;     // pin digital 11 de Arduino a IN4 de modulo controlador
int demora = 10;


//**Motor secado**
int IN5 = 4;      // pin digital 8 de Arduino a IN1 de modulo controlador
int IN6 = 5;      // pin digital 9 de Arduino a IN2 de modulo controlador
int IN7 = 6;     // pin digital 10 de Arduino a IN3 de modulo controlador
int IN8 = 7;     // pin digital 11 de Arduino a IN4 de modulo controlador


//**LEDS FUNCIONES DE LA LAVADORA
int lavado = 22;
int enjuague = 24;
int secado = 26;
int listo = 28;


//**LEDS ENTRADAS Y SALIDAS
int agua_in = 30;
int agua_out = 32;
int aire_caliente = 34;

//lcd
#include<LiquidCrystal_I2C.h>
#include<LiquidMenu.h>
LiquidCrystal_I2C lcd(0x27,16,2);

//Encoder 
#define outputA 3
#define outputB 2
#define sw 12

int aState;
int aLastState;

//Menu principal 
LiquidLine linea1(1, 0, "Lavado");
LiquidLine linea2(1, 1, "Secado");
LiquidLine linea3(1, 0, "Conectar dispositivo");
LiquidScreen pantalla1(linea1,linea2,linea3);

//Menu lavado
LiquidLine linea1_2(1, 0, "Lavar");
LiquidLine linea2_2(1, 1, "Enjuagar");
LiquidLine linea3_2(1, 0, "Lavado Completo");
LiquidLine linea4_2(1, 1, "atras");
LiquidScreen pantalla2(linea1_2,linea2_2,linea3_2,linea4_2);


//Menu Secado
LiquidLine linea1_3(1, 0, "10 min");
LiquidLine linea2_3(1, 1, "20 min");
LiquidLine linea3_3(1, 0, "30 min");
LiquidLine linea4_3(1, 1, "Atras");
LiquidScreen pantalla3(linea1_3,linea2_3,linea3_3,linea4_3);


LiquidMenu menu(lcd,pantalla1,pantalla2,pantalla3);


void setup() {
  //Motor Lavado
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  //Motor Secado
  pinMode(IN5, OUTPUT);
  pinMode(IN6, OUTPUT);
  pinMode(IN7, OUTPUT);
  pinMode(IN8, OUTPUT);
  //Declaracion de los leds como salidas
  pinMode(lavado, OUTPUT);
  pinMode(enjuague, OUTPUT);
  pinMode(secado, OUTPUT);
  pinMode(listo, OUTPUT);
  pinMode(agua_in, OUTPUT);
  pinMode(agua_out, OUTPUT);
  pinMode(aire_caliente, OUTPUT);

  //LCD
  lcd.init();
  lcd.backlight();

  pinMode(sw,INPUT_PULLUP);
  menu.init();


  //Menu principal
  linea1.set_focusPosition(Position::LEFT); 
  linea2.set_focusPosition(Position::LEFT); 
  linea3.set_focusPosition(Position::LEFT); 

  linea1.attach_function(1, lavado_final); //IMPLEMENTAR FUNCION
  linea2.attach_function(1, secado_final);//IMPLEMENTAR FUNCION
  linea3.attach_function(1, conectar_dispositivo); //IMPLEMENTAR FUNCION
  menu.add_screen(pantalla1);

  //Menu lavado
  linea1_2.set_focusPosition(Position::LEFT); 
  linea2_2.set_focusPosition(Position::LEFT); 
  linea3_2.set_focusPosition(Position::LEFT); 
  linea4_2.set_focusPosition(Position::LEFT); 
   
  linea1_2.attach_function(1, lavar); //IMPLEMENTAR FUNCION
  linea2_2.attach_function(1, enjuagado);//IMPLEMENTAR FUNCION
  linea3_2.attach_function(1, lavado_completo);//IMPLEMENTAR FUNCION
  linea4_2.attach_function(1, atras);//IMPLEMENTAR FUNCION
  menu.add_screen(pantalla2);

  //Menu Enjuagado 
  linea1_3.set_focusPosition(Position::LEFT); 
  linea2_3.set_focusPosition(Position::LEFT); 
  linea3_3.set_focusPosition(Position::LEFT); 
  linea4_3.set_focusPosition(Position::LEFT); 
   
  linea1_3.attach_function(1, secar); //IMPLEMENTAR FUNCION
  linea2_3.attach_function(1, secar);//IMPLEMENTAR FUNCION
  linea3_3.attach_function(1, secar);//IMPLEMENTAR FUNCION
  linea4_3.attach_function(1, atras);//IMPLEMENTAR FUNCION
  menu.add_screen(pantalla3);   

  pantalla1.set_displayLineCount(2);
  pantalla2.set_displayLineCount(2);
  pantalla3.set_displayLineCount(2);
  menu.set_focusedLine(0);

  menu.update();
}
int incremento=0;
void loop() {
    selectOption();

  aState = digitalRead(outputA); 
    if (aState != aLastState){     
      if (digitalRead(outputB) != aState) { 
        incremento++;
        if(incremento>1){
          incremento=0; 
          menu.switch_focus(false);}
      } else {
        incremento++;
        if(incremento>1){
          incremento=0; 
          menu.switch_focus(true);}
      }
      menu.update();
      aLastState = aState;
  }
}

void conectar_dispositivo(){}




void selectOption(){
  if(digitalRead(sw) == LOW){
      menu.call_function(1);
      delay(500);
  }
}

void lavado_final(){
  menu.change_screen(2);
  menu.set_focusedLine(0);
  }

void secado_final(){
  menu.change_screen(3);
  menu.set_focusedLine(0);
  }
void atras(){
  menu.change_screen(1);
  menu.set_focusedLine(0);
}
void lavar() {
  lcd.clear();
  lcd.print("Lavando");
  digitalWrite(lavado, HIGH); //DEBERIA ENCENDERSE CON EL ENCODER***FUNCION DEL ENCODER ROTATIVO
  digitalWrite(agua_in, HIGH);//Simula el llenado de agua
  delay(15000);
  digitalWrite(agua_in, LOW);
  lavar_motor();
  digitalWrite(lavado, LOW);
  digitalWrite(listo, HIGH);
  lcd.clear();
  lcd.print("**LAVADO LISTO**");
  delay(10000);
  digitalWrite(listo, LOW);
  atras();
}
void enjuagado() {
  lcd.clear();
  lcd.print("Enjuagando...");
  digitalWrite(enjuague, HIGH); //DEBERIA ENCENDERSE CON EL ENCODER***FUNCION DEL ENCODER ROTATIVO
  digitalWrite(agua_out, HIGH);//El agua está siendo expulsada
  delay(10000);
  enjuagar_motor();
  digitalWrite(enjuague, LOW);
  digitalWrite(agua_out, LOW);
  digitalWrite(listo, HIGH);
  lcd.clear();
  lcd.print("**ENJUAGUE LISTO**");
  delay(10000);
  digitalWrite(listo, LOW);
  atras();


}
void lavado_completo() {
  digitalWrite(lavado, HIGH); //DEBERIA ENCENDERSE CON EL ENCODER***FUNCION DEL ENCODER ROTATIVO
  digitalWrite(enjuague, HIGH); //DEBERIA ENCENDERSE CON EL ENCODER***FUNCION DEL ENCODER ROTATIVO
  lavar();
  enjuagado();

}
void secar() { //los ciclos seran 10 ,20 o 30 min
  lcd.clear();
  lcd.print("SECANDO...");
  digitalWrite(secado, HIGH); //ESTO VA EN EL ENCODER
  for (int i = 0; i < 10 / 2; i++)
  {
    digitalWrite(aire_caliente, HIGH);
    secado_motor();
    delay(3000);
    digitalWrite(aire_caliente, LOW);
    secado_motor();
    delay(3000);
    i++;
  }
  digitalWrite(secado, LOW);
  digitalWrite(listo, HIGH);
  lcd.clear();
  lcd.print("**SECADO LISTO**");
  delay(10000);
  digitalWrite(listo, LOW);
  atras();

}
//El motor1 Gira 90 grados en sentido horario y luego 90 en sentido antihorario por 10 ciclos
void lavar_motor() {
  int contador = 0;
  while (contador < 5) {
    //Giro de 90 grados en sentido horario
    for (int i = 0; i < 128; i++)
    {
      digitalWrite(IN1, HIGH);  // paso 1
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      delay(demora);

      digitalWrite(IN1, LOW); // paso 2
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      delay(demora);

      digitalWrite(IN1, LOW); // paso 3
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
      delay(demora);

      digitalWrite(IN1, LOW); // paso 4
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
      delay(demora);
    }
    delay(1000);

    //Giro de 90 grados en sentido antihorario
    for (int i = 0; i < 128; i++)
    {
      digitalWrite(IN1, LOW); // paso 4
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
      delay(demora);

      digitalWrite(IN1, LOW); // paso 3
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
      delay(demora);

      digitalWrite(IN1, LOW); // paso 2
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      delay(demora);

      digitalWrite(IN1, HIGH);  // paso 1
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      delay(demora);
    }
    delay(1000);
    contador++;
  }
}


//El motor2 gira en sentido horario sin detenerse por 5 min
void enjuagar_motor() {
  int contador = 0;
  int y = 4;
  while (contador < 5) {
    //Giro de 90 grados en sentido horario
    for (int i = 0; i < 512; i++)
    {
      digitalWrite(IN1, HIGH);  // paso 1
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      delay(demora / y);

      digitalWrite(IN1, LOW); // paso 2
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      delay(demora / y);

      digitalWrite(IN1, LOW); // paso 3
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
      delay(demora / y);

      digitalWrite(IN1, LOW); // paso 4
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
      delay(demora / y);
    }
    contador++;
  }
}

//el motor girara sin detenerse durante los ciclos determinados
void secado_motor() {
  int contador = 0;
  int y = 3;
  while (contador < 2) {
    //Giro de 90 grados en sentido horario
    for (int i = 0; i < 512; i++)
    {
      digitalWrite(IN5, HIGH);  // paso 1
      digitalWrite(IN6, LOW);
      digitalWrite(IN7, LOW);
      digitalWrite(IN8, LOW);
      delay(demora / y);

      digitalWrite(IN5, LOW); // paso 2
      digitalWrite(IN6, HIGH);
      digitalWrite(IN7, LOW);
      digitalWrite(IN8, LOW);
      delay(demora / y);

      digitalWrite(IN5, LOW); // paso 3
      digitalWrite(IN6, LOW);
      digitalWrite(IN7, HIGH);
      digitalWrite(IN8, LOW);
      delay(demora / y);

      digitalWrite(IN5, LOW); // paso 4
      digitalWrite(IN6, LOW);
      digitalWrite(IN7, LOW);
      digitalWrite(IN8 , HIGH);
      delay(demora / y);
    }
    contador++;
  }
}
