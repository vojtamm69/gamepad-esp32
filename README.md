

// Knihovny: 
#include <Arduino.h>
#include <BleGamepad.h>
 
 
//ABXY TLACITKA:
 
#define X_BUTTON 26         // A (Pin G26)
 
#define CIRCLE_BUTTON 25   // B (Pin G25)
 
#define TRIANGLE_BUTTON 33  // Y  (Pin G33)
 
#define SQUARE_BUTTON 32    // X  (Pin G32)
 
 
// Levý joystick: 
#define LEFT_VRX_JOYSTICK 34 // Pohyb na souřadnici X (zleva do prava)
 
#define LEFT_VRY_JOYSTICK 35  // Pohyb na souřadnici Y (Zhora dolů)
 
/*
  Pravý joystick nepoužijeme, ale tady je kod pro něj:

  #define RIGHT_VRX_JOYSTICK 0
 
  #define RIGHT_VRY_JOYSTICK 0
 
  #define NUM_BUTTONS 0

*/

 
int buttonsPins[NUM_BUTTONS] = {X_BUTTON, CIRCLE_BUTTON, TRIANGLE_BUTTON, SQUARE_BUTTON,
 
                          R1_BUTTON, R2_BUTTON, L1_BUTTON, L2_BUTTON,
 
                          START_BUTTON, SELECT_BUTTON, PS_BUTTON,
 
                          R3_BUTTON, L3_BUTTON};
 
 
int androidGamepadButtons[NUM_BUTTONS] = {1, 2, 3, 4, 8, 10, 7, 9, 12, 11, 13, 15, 14};
 
int PS1GamepadButtons[NUM_BUTTONS] = {2, 3, 4, 1, 6, 8, 5, 7, 10, 9, 13, 12, 11};
 
int PCGamepadButtons[NUM_BUTTONS] = {1, 2, 4, 3, 6, 8, 5, 7, 10, 9, 0, 12, 11};
 
 
uint16_t leftVrxJoystickLecture = 0;
 
uint16_t leftVryJoystickLecture = 0;
 
uint16_t rightVrxJoystickLecture = 0;
 
uint16_t rightVryJoystickLecture = 0;
 
uint16_t leftVrxJoystickValue = 0;
 
uint16_t leftVryJoystickValue = 0;
 
uint16_t rightVrxJoystickValue = 0;
 
uint16_t rightVryJoystickValue = 0;
 
 
typedef enum{ANDROID, PS1, PC} GamepadModes;
 
GamepadModes gamepadMode = ANDROID;
 
 
BleGamepad bleGamepad("Maker101 Gamepad", "Maker101 Home");
 
BleGamepadConfiguration bleGamepadConfig;  
 
 
void setup() {
 
  // Setup kod:
 
  delay(1000);
 
  Serial.begin(115200);
 
  for(int i=0; i<NUM_BUTTONS; i++){
 
    pinMode(buttonsPins[i], INPUT_PULLUP);
 
  }
 
  bleGamepadConfig.setAutoReport(false);
 
  bleGamepadConfig.setControllerType(CONTROLLER_TYPE_GAMEPAD); 
 
  bleGamepadConfig.setVid(0xe502);
 
  bleGamepadConfig.setPid(0xabcd);
 
  bleGamepadConfig.setHatSwitchCount(4);
 
  bleGamepad.begin(&bleGamepadConfig);
 
}
 
void loop() {
 
  // Hlavní část kodu která se bude opakovat:
 
  if(bleGamepad.isConnected()){
 
    
 
    leftVrxJoystickLecture = analogRead(LEFT_VRX_JOYSTICK);
 
    leftVryJoystickLecture = analogRead(LEFT_VRY_JOYSTICK);
 
    rightVrxJoystickLecture = analogRead(RIGHT_VRX_JOYSTICK);
 
    rightVryJoystickLecture = analogRead(RIGHT_VRY_JOYSTICK);
 
    
 
    leftVrxJoystickValue = map(leftVrxJoystickLecture, 4095, 0, 0, 32737);
 
    leftVryJoystickValue = map(leftVryJoystickLecture, 0, 4095, 0, 32737);
 
    rightVrxJoystickValue = map(rightVrxJoystickLecture, 4095, 0, 0, 32737);
 
    rightVryJoystickValue = map(rightVryJoystickLecture, 0, 4095, 0, 32737);
 
   
  // Mody gamepadu: 
    switch(gamepadMode){
 
      case ANDROID:
 
        for(int i=0; i<NUM_BUTTONS; i++){
 
          if(!digitalRead(buttonsPins[i])){
 
              bleGamepad.press(androidGamepadButtons[i]);  
 
          }
 
          else{
 
              bleGamepad.release(androidGamepadButtons[i]);    
 
          }
 
          joysticksHandlerForMobile(leftVrxJoystickValue, leftVryJoystickValue, rightVrxJoystickValue, rightVryJoystickValue);
 
        }
 
        break;
 
      case PS1:
 
        for(int i=0; i<NUM_BUTTONS; i++){
 
          if(!digitalRead(buttonsPins[i])){
 
            bleGamepad.press(PS1GamepadButtons[i]);    
 
          }
 
          else{
 
            bleGamepad.release(PS1GamepadButtons[i]);      
 
          }
 
          joysticksHandlerForMobile(leftVrxJoystickValue, leftVryJoystickValue, rightVrxJoystickValue, rightVryJoystickValue);
 
        }
 
        break;
 
        case PC:
 
          for(int i=0; i<NUM_BUTTONS; i++){
 
            if(!digitalRead(buttonsPins[i])){
 
              bleGamepad.press(PCGamepadButtons[i]);
 
            }
 
            else{
 
              bleGamepad.release(PCGamepadButtons[i]);
 
            }
 
            joysticksHandlerForPC(leftVrxJoystickValue, leftVryJoystickValue, rightVrxJoystickValue, rightVryJoystickValue);
 
          }
 
          break;
 
    }
 
    bleGamepad.sendReport();
 
  }
 
}
 
void joysticksHandlerForMobile(uint16_t leftVrx, uint16_t leftVry, uint16_t rightVrx, uint16_t rightVry){
 
  bleGamepad.setLeftThumb(leftVrx, leftVryJoystickValue);
 
  bleGamepad.setRightThumb(rightVrxJoystickValue, rightVryJoystickValue);  
 
}
 
void joysticksHandlerForPC(uint16_t leftVrx, uint16_t leftVry, uint16_t rightVrx, uint16_t rightVry){
 
  bleGamepad.setX(leftVrxJoystickValue);
 
  bleGamepad.setY(leftVryJoystickValue);
 
  bleGamepad.setZ(rightVrxJoystickValue);
 
  bleGamepad.setRX(rightVryJoystickValue);
 
}
