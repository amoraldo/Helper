#Buenas prácticas Arduino

###Organizacion de archivos

#####Documentos necesarios a la hora de desarrollar un proyecto con Arduino

-	Planilla de componentes y costos: Ira reflejado los materiales necesarios con un estimativo de costos con fecha de última actualización (se pueden usar links de MercadoLibre, Nubbeo, locales TDF o Bs As). También debe especificarse si se necesita algún trabajo tercerizado: Fabricación de PCB, matriz, mecanizado de piezas, etc.

#####Código:

- Pines.h: contiene las definiciones de los pines con su descripción en el comentario
Se debe incluir el modelo de placa que se utilizará de manera de dejar en claro sin duda cual es el hardware que se utilizó a la hora del diseño. En caso de ser compatible con varios modelos de placa, puede definirse distintas definiciones para distintos modelos.

	Ejemplo:

      // STM32F103T8R8
      #if defined (ARDUINO_ARCH_STM32F1)

        #define RELAY1        PD2 // Salida de relay comando sentido de giro
        #define RELAY2        PB5 // Salida de relay activacion de lamparas
        #define KEY_COL0      PA5 // Filas y columnas de matriz de teclado
        #define KEY_COL1      PA6
        #define KEY_COL2      PA7
        #define KEY_R0        PC10
        #define KEY_R1        PC11
        #define KEY_R2        PC12
      // LCD
        #define   RS  PB4 // Reset pin
        #define   EN  PB3 // Enabled pin
        #define   DD4 PB9 // 4 bits de datos
        #define   DD5 PB8
        #define   DD6 PB7
        #define   DD7 PB6
      #else
        #error Verifique los pines de su tarjeta antes de continuar.
      #endif

- Settings.h: contiene las definiciones de los seteos con su descripción en el comentario
Esto permite que puedan modificarse fácilmente los valores iniciales, rangos, limites, tiempos, etc sin acudir al código principal para hacerlo 

  Ejemplo:


    #define Version       1.0    // Version del equipo

    //LCD Menu
    #define LCD_ROW       4      // Cantidad de filas LCD 
    #define LCD_COL       16     // Cantidad de Columnas LCD
    #define REFRESH       50     // update display
    #define BLINKING      1000   // update Blinking LED,sound, backlight, etc
    #define T_Relay       3000    // Tiempo de activacion de relay

    //Limites
    #define PWM10p        25     // Valor de PWM al 10% [0;255]
    #define PWM90p        230    // Valor de PWM al 90% [0;255]
    #define press_min     2000    // valores analogicos de presion
    #define press_max     2700

    //Constantes útiles

    // result values
    #define Fail 0
    #define Pass 1
    #define Continue 2
    #define Next 3
    #define Complete 4
    #define End 10

    #define BLANK "                    "  // Blank define 20 caracteres vacios en LCD
    #define _Press LOW                   // Define la logica de los botones
    #define _NPress HIGH
    #define POS LOW
    #define NEG HIGH


- Funciones.cpp: Contiene las funciones estándar utilizadas en el proyecto, de manera que el código principal quede lo mas limpio y legible posible. Algunas funciones estándar podrían ser: SetupPins(), SetupLCD(), WriteLCD(), etc
Notar que deben usarse los “include” a los archivos “pines.h” y “settings.h” además del propio “funciones.h”
Notar también, que no deben repetirse las llamadas a hardware en más de un archivo. Por ejemplo, si hemos optado por colocar las funciones del LCD en “funciones.h”, desde otros archivos haremos uso de las funciones LCD que solo se encontran en “funciones.h”. 
Ejemplo:


    #include <Arduino.h>
    #include "pines.h"
    #include "settings.h"
    #include "funciones.h"

    #include <LiquidCrystal.h>
    LiquidCrystal lcd(RS, EN, DD4, DD5, DD6, DD7); //Initialize the LCD

    extern byte Step;
    extern char result;
    String LCD_Row1;
    String LCD_Row2;
    String LCD_Row3;
    String LCD_Row4;

    void Setup_lcd(){
      lcd.begin(LCD_COL, LCD_ROW);    
    }

    // LCD_ROWx     = "12345678901234567890";
    String LCD_Row1 = BLANK;
    String LCD_Row2 = BLANK;
    String LCD_Row3 = BLANK;
    String LCD_Row4 = BLANK;
    String oldLCD_Row1 = BLANK;
    String oldLCD_Row2 = BLANK;
    String oldLCD_Row3 = BLANK;
    String oldLCD_Row4 = BLANK;

    void lcdRefresh(){
      if(oldLCD_Row1 != LCD_Row1){
        lcd.setCursor(0, 0);
        lcd.print(LCD_Row1);
        oldLCD_Row1 = LCD_Row1;
      }
      if(oldLCD_Row2 != LCD_Row2){
        lcd.setCursor(0, 1);
        lcd.print(LCD_Row2);
        oldLCD_Row2 = LCD_Row2;
      }
      if(oldLCD_Row3 != LCD_Row3){
        lcd.setCursor(LCD_COL, 0);
        lcd.print(LCD_Row3);
        oldLCD_Row3 = LCD_Row3;
      }
      if(oldLCD_Row4 != LCD_Row4){
        lcd.setCursor(LCD_COL, 1);
        lcd.print(LCD_Row4);
        oldLCD_Row4 = LCD_Row4;
      }
    }
    void Setup_pins(){
      pinMode(RELAY1,OUTPUT);
      pinMode(RELAY2,OUTPUT);
      pinMode(KEY_COL0,OUTPUT);
      pinMode(KEY_COL1,OUTPUT);
      pinMode(KEY_COL2,OUTPUT);
      pinMode(KEY_R0,INPUT);
      pinMode(KEY_R1,INPUT);
      pinMode(KEY_R2,INPUT);

      digitalWrite(RELAY1,LOW);
      digitalWrite(RELAY2,LOW);
      digitalWrite(KEY_COL0,LOW);
      digitalWrite(KEY_COL1,LOW);
      digitalWrite(KEY_COL2,LOW);
    }

- Funciones.h: Contiene la declaración de las funciones utilizadas en Funciones.cpp


    #ifndef FUNCIONES_H
      #define FUNCIONES_H

      void Setup_lcd();
      void lcdRefresh();
      void Setup_pins();

    #endif

- main.ino: Contiene el flujo principal del código fuente. El mismo debe ser legible y facil de interpretar, haciendo llamadas a librerias especificas de cada tecnología. Hará buen uso de variables externas para compartir informacion vital con otras funciones declaradas en otros archivos, evitando el guardado innecesario de variables temporales u ocasionales.


    // Autor: Andres Moraldo
    // Descripción: Testeo de placa PCB para produccion

    #include <Arduino.h>
    #include "pines.h"
    #include "settings.h"
    #include "funciones.h"
    #include "tests.h"

    // LCD_ROWx     = "12345678901234567890";
    byte Step=1;
    char result=Next;
    unsigned long lastTime=0;
    extern String LCD_Row1;
    extern String LCD_Row2;
    extern String LCD_Row3;
    extern String LCD_Row4;
    int i=0;

void setup() {
      delay(200);
      Serial.begin(115200);
      delay(1000);
      Serial.println("");
      Serial.println("Secuencia de testeo para PCB respirador. V" + String(Version));
      Serial.println("Presion OK para iniciar");
      delay(200);
      disableDebugPorts();
      Setup_pins();
      Setup_lcd();
      LCD_Row1 = "TestPCB V" + String(Version) + "        ";
	  LCD_Row2 = "Presione OK         ";
      LCD_Row3 = " para iniciar...    "; 
	  LCD_Row4 = BLANK;
      lcdRefresh();
      delay(1500);
      lastTime=millis();
    }

    void loop() {
      if (millis() > lastTime + REFRESH){  //refresh lcd
        lcdRefresh();
        lastTime=millis();    
      }
	  LCD_Row1 = "Prueba de tiempo";
      LCD_Row2 = "     " + str(i) + "mS";
      LCD_Row3 = BLANK; 
	  LCD_Row4 = BLANK;
	  i++;
	  delay(1);     // mSeg
      }
    }

#####Esquemático:
Realizar un archivo esquematico del hardware implementado, donde por etapas pueda construirse y replicarse el dispositivo

#####Diseños 3d:
En los casos que se realicen diseños 3d de carcazas o componentes, incluir los archivos STL junto con un archivo de texto con la configuracion necesaria para imprimir (altura de capa, cantidad de relleno, etc). Ademas, incluir archivo de diseño (SKP en el caso de sketchup)
