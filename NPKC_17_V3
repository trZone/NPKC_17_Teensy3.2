/* Buttons to USB Keyboard Example

   You must select Keyboard from the "Tools > USB Type" menu

   This example code is in the public domain.
*/

#include <Keypad.h>
#include <EEPROM.h>
#include <stdlib.h>

const byte ROWS = 5; //four rows
const byte COLS = 4; //three columns

byte rowPins[ROWS] = {12, 11, 10, 9, 8}; //connect to the row pinouts of the keypad
byte colPins[COLS] = {4, 3, 2, 1}; //connect to the column pinouts of the keypad


//matrix layout numbered in chars to reference later in layers as ints
//matrix has 20 locations but there are physically only 17 buttons
//the subroutine keyCharToLayer returns an int from 0 to 16 for the 17 buttons
//which are remembered in the array called keyLayers
  char keys[ROWS][COLS] = {
  {'0', '1', '2', '3'},
  {'4', '5', '6', '7'},
  {'8', '9', 'a', 0},
  {'b', 'c', 'd', 0},
  {'e', 'f', 'g', 0}
};

int layer = 0; //LAYER 0 numpad keys, LAYER 1 normal keys
int keyHolds[6] = {0, 0, 0, 0, 0}; //used for 6-key rollover, remembers button numbers that are held
boolean funLockHeld[4] = { false, false, false, false };

char inData[20]; // Allocate some space for the string
char inChar; // Where to store the character read
byte tIndex = 0; // Index into array; where to store the character


//define LED pins on the board in order
const int ledPins[4] = {20, 21, 22, 23};

//num lock = lock     = blue   -> pink
//10 tails = divide   = purple -> 
//heart    = multiply = red    -> blue
//nerv     = minus    = pink   -> red


unsigned long startTime = millis();

Keypad kpd = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );

//matrix has 20 locations but there are physically only 17 buttons
//I give the 1st key, numlock a zero so Keyboard.send_now() won't send it during main loop
char keyLayers[2][17] = {
  
   //LAYER 0 numpad keys. btw, numlock is 0x53
  {0, 0x54, 0x55, 0x56,
   0x5F, 0x60, 0x61, 0x57,
   0x5C, 0x5D, 0x5E,
   0x59, 0x5A, 0x5B,
   0x62, 0x63, 0x58},
   
   //LAYER 1 normal keys
  {0, 0x2F, 0x2A, KEY_MINUS,
   '7', '8', '9', '+',
   '4', '5', '6',
   '1', '2', '3',
   '0', '.', 10,},
   
};


uint8_t keyNone[8] = { 0, 0, 0, 0, 0, 0, 0, 0 };
uint8_t keyA[8] = { 0, 0, 4, 0, 0, 0, 0, 0 };

//digitalwrite(ledPin, HIGH) is 0xFF. I am using PWM pins which take any value from 0x00 to 0xFF
//to control brightness because one of my LEDs is purple and its not as bright as the others
int ledHIGHS[4] = { 0x3F, 0xFF, 0x3F, 0x3F };

void setup() {

  pinMode(ledPins[0], OUTPUT);
  pinMode(ledPins[1], OUTPUT);
  pinMode(ledPins[2], OUTPUT);
  pinMode(ledPins[3], OUTPUT);
  
  delay(2000);
  Serial.begin(9600);
  Mouse.begin();
  Keyboard.begin();
  
  //get layer from saved data
  int checkLayer = EEPROM.read(0);
  if ( checkLayer > 5 ) { layer = 0; }
  else { layer = checkLayer; }

  kpd.setDebounceTime(1);
  
  activateLedMode();
  
//  keyTest();

}



void loop() {
      

  //the one second loop counter
  if ( (millis()-startTime)>1000 ) {
    startTime = millis();
    //any routines that you want evey 1 second
  }
  
  
  if (kpd.getKeys())
  {
    
    int key = kpd.getKey();
    
    for (int i=0; i<LIST_MAX; i++)   // Scan the whole key list.
    {
      if ( kpd.key[i].stateChanged )   // Only find keys that have changed state.
      {
        switch (kpd.key[i].kstate) {  // Report active key state : IDLE, PRESSED, HOLD, or RELEASED
            case PRESSED:
              if ( kpd.key[i].kchar == '0' ) { funLockHeld[0] = true; Serial.println("ZERO HELD [numlock], press another key for layer"); }
              else { pressKey(keyCharToLayer(kpd.key[i].kchar)); } //subroutine handles pressing keys
              break;
            case HOLD:
              if (( layer != 0 ) && ( funLockHeld[0] == 0 )) {
                logHold(keyCharToLayer(kpd.key[i].kchar)); //subroutine handles holding keys
              }
              break;
            case RELEASED:
              if ( kpd.key[i].kchar == '0' ) { funLockHeld[0] = false; Serial.println("ZERO RELEASED [numlock]"); }
              else if ( funLockHeld[0] == false ) { releaseKey(keyCharToLayer(kpd.key[i].kchar)); } //subroutine handles releasing keys
             // releaseKey(kpd.key[i].kchar); //subroutine handles releasing keys
             // break;
           // case IDLE: //no current need to handle idle keys
        }
      }
    }
  }
  

  //LAYER 0 numpad keys
  //we have to do 6-key rollover manually. which means sending the keys every cycle
  if ( layer == 0 ) {
    
     Keyboard.set_key1(keyLayers[layer][keyHolds[0]]);
     Keyboard.set_key2(keyLayers[layer][keyHolds[1]]);
     Keyboard.set_key3(keyLayers[layer][keyHolds[2]]);
     Keyboard.set_key4(keyLayers[layer][keyHolds[3]]);
     Keyboard.set_key5(keyLayers[layer][keyHolds[4]]);
     Keyboard.set_key6(keyLayers[layer][keyHolds[5]]);
     Keyboard.send_now();
   
  }

}




//subroutine handles pressing keys
void pressKey(int sendKey) {

  //change layer if numlock held then slash is pressed
  if (( funLockHeld[0] == true ) && ( sendKey == 1 )) {
  //if ( sendKey == 0 ) {
    if ( layer == 0 ) { activateKeyMode(1); }
    else if ( layer == 1 ) { activateKeyMode(0); }
  }
  else
  {
     //LAYER 1 normal keys
     if ( layer == 1 ) {
       Keyboard.press(keyLayers[layer][sendKey]);
     }
     //LAYER 0 numpad keys
     else if ( layer == 0 ) {

       int sendKeyInt = digit_to_int(sendKey);
       logHold(sendKey);
       
     }   
 
  }

}

//6-key rollover: subroutine remembers hold events
void logHold(int sendKey) {

  //check for an available logical key to remebmer the key number  
  if ( keyHolds[0] == 0 ) { keyHolds[0] = sendKey; }
  else if ( keyHolds[1] == 0 ) { keyHolds[1] = sendKey; }
  else if ( keyHolds[2] == 0 ) { keyHolds[2] = sendKey; }
  else if ( keyHolds[3] == 0 ) { keyHolds[3] = sendKey; }  
  else if ( keyHolds[4] == 0 ) { keyHolds[4] = sendKey; }  
  else if ( keyHolds[5] == 0 ) { keyHolds[5] = sendKey; } 
  
  //if we are already remembering 6 (user is now holding 7 keys), forget [release] the first key
  //that the user pressed. use this section if you want it. it will allow a seventh key to be
  //pressed but it will forget the first key. keep in mind this is only implemented for numpad keys
  //else {
  //  keyHolds[0] = keyHolds[1];
  //  keyHolds[1] = keyHolds[2];
  //  keyHolds[2] = keyHolds[3];
  //  keyHolds[3] = keyHolds[4];
  //  keyHolds[4] = keyHolds[5];
  //  keyHolds[5] = sendKey;
  //}

  //this is debug output. can be removed or commented.
  Serial.print("   HOLD SENT: ");  
  Serial.println(sendKey);
  Serial.print("     1: ");
  Serial.println(keyHolds[0]);
  Serial.print("     2: ");
  Serial.println(keyHolds[1]);
  Serial.print("     3: ");
  Serial.println(keyHolds[2]);
  Serial.print("     4: ");
  Serial.println(keyHolds[3]);
  Serial.print("     5: ");
  Serial.println(keyHolds[4]);
  Serial.print("     6: ");
  Serial.println(keyHolds[5]);
  
}

//6-key rollover: subroutine forgets hold events if user releases a key
void logRelease(int sendKey) {
  
  int released = 6;
  
  //find where the key was remembered and make it zero
  if ( keyHolds[0] == sendKey ) { released = 0; }
  else if ( keyHolds[1] == sendKey ) { released = 1; }
  else if ( keyHolds[2] == sendKey ) { released = 2; }
  else if ( keyHolds[3] == sendKey ) { released = 3; }  
  else if ( keyHolds[4] == sendKey ) { released = 4; }  
  else if ( keyHolds[5] == sendKey ) { released = 5; } 

  //a remembered key was found and released. we can't have a zero in the middle, move it to the bottom
  if ( released != 6 ){
      if ( released == 0 ) {
        keyHolds[0] = keyHolds[1];
        keyHolds[1] = keyHolds[2];
        keyHolds[2] = keyHolds[3];
        keyHolds[3] = keyHolds[4];
        keyHolds[4] = keyHolds[5];
        keyHolds[5] = 0;
      }
      else if ( released == 1 ) {
        keyHolds[1] = keyHolds[2];
        keyHolds[2] = keyHolds[3];
        keyHolds[3] = keyHolds[4];
        keyHolds[4] = keyHolds[5];
        keyHolds[5] = 0;
      }
      else if ( released == 2 ) {
        keyHolds[2] = keyHolds[3];
        keyHolds[3] = keyHolds[4];
        keyHolds[4] = keyHolds[5];
        keyHolds[5] = 0;
      }
      else if ( released == 3 ) {
        keyHolds[3] = keyHolds[4];
        keyHolds[4] = keyHolds[5];
        keyHolds[5] = 0;
      }
      else if ( released == 4 ) {
        keyHolds[4] = keyHolds[5];
        keyHolds[5] = 0;
      }
      else if ( released == 5 ) {
        keyHolds[5] = 0;
      } 

  }
  
  
  //this is debug output. can be removed or commented.
  Serial.print("   RELEASE SENT: ");  
  Serial.println(sendKey);
  Serial.print("     1: ");
  Serial.println(keyHolds[0]);
  Serial.print("     2: ");
  Serial.println(keyHolds[1]);
  Serial.print("     3: ");
  Serial.println(keyHolds[2]);
  Serial.print("     4: ");
  Serial.println(keyHolds[3]);
  Serial.print("     5: ");
  Serial.println(keyHolds[4]);
  Serial.print("     6: ");
  Serial.println(keyHolds[5]);
  
  
}


//subroutine handles releasing keys
void releaseKey(int sendKey) {
  
  if ( sendKey == '0' ) {

  }
  else if (( funLockHeld[0] == true ) && ( sendKey == 1 )) {

  }
  else
  {
     //LAYER 1 normal keys
     if ( layer == 1 ) {
       Keyboard.release(keyLayers[layer][sendKey]);
       
     }//
     //LAYER 0 numpad keys
     else if ( layer == 0 ) {
       //Keyboard.set_key1(0);
       //Keyboard.send_now();
     }   
     logRelease(sendKey);

  }

}


//this is for converting a char to int. I tried making keys[ROWS][COLS] an int so that
//every subroutine used same numbers, example keyHolds[i], but
//some subroutines failed to transfer the int variable to commands
int digit_to_int(char d)
{
 char str[2];
 str[0] = d;
 str[1] = '\0';

 return (int)strtol(str, (char **)NULL, 10);
}


int keyCharToLayer(char kChar) {
  int kInt;

  if ( kChar == '0' ) { kInt = 0; }
  else if ( kChar == '1' ) { kInt = 1; }
  else if ( kChar == '2' ) { kInt = 2; }
  else if ( kChar == '3' ) { kInt = 3; }
  else if ( kChar == '4' ) { kInt = 4; }
  else if ( kChar == '5' ) { kInt = 5; }
  else if ( kChar == '6' ) { kInt = 6; }
  else if ( kChar == '7' ) { kInt = 7; }
  else if ( kChar == '8' ) { kInt = 8; }
  else if ( kChar == '9' ) { kInt = 9; }
  else if ( kChar == 'a' ) { kInt = 10; }
  else if ( kChar == 'b' ) { kInt = 11; }
  else if ( kChar == 'c' ) { kInt = 12; }
  else if ( kChar == 'd' ) { kInt = 13; }
  else if ( kChar == 'e' ) { kInt = 14; }
  else if ( kChar == 'f' ) { kInt = 15; }
  else if ( kChar == 'g' ) { kInt = 16; }
  
    //this is debug output. can be removed or commented.
    Serial.print("received char: ");
    Serial.print(kChar);
    Serial.print(" = button # ");
    Serial.print(kInt);
    Serial.print(' ');
    Serial.print(" layer ");
    Serial.print(layer);
    Serial.print(" = ");
    Serial.print(keyLayers[layer][kInt]);
    Serial.println();

 //return the integer
 return kInt;
  
}


//when layer is changed
void activateKeyMode(int newLayer) {
 layer = newLayer;
 for (int kh = 0; kh<6; kh++) {
   keyHolds[kh] = 0;
 }
 EEPROM.write(0, newLayer);
 activateLedMode();

}



//NOTE: use digitalWrite with HIGH or LOW if you aren't using PWM pins for brightness control
void activateLedMode() {
  
  //turn off all of the leds
  for (int i=0; i<5; i++) {
    analogWrite(ledPins[i], 0x00);
  }
  
  //turn on the led for the layer
  analogWrite(ledPins[layer], ledHIGHS[layer]);

  
}
