# Arduino Keypad Login System

## Description
This project implements a login system using an Arduino, a 4x4 keypad, an LCD 16x2 display, and a buzzer. The system verifies user input against a predefined password and provides feedback through the buzzer and display.

## Components
- Arduino Board
- 4x4 Keypad
- LCD 16x2
- Buzzer
- Connecting Wires

## Pin Definitions
- **Keypad**
  - Rows: Pins 2, 3, 4, 5
  - Columns: Pins 6, 7, 8, 9
- **LCD**
  - RS: A5
  - E: A4
  - D4: A3
  - D5: A2
  - D6: A1
  - D7: A0
- **Buzzer**: Pin 11

## Password and Attempts
- **Password Length**: 5 characters
- **Correct Password**: `123AB`
- **Max Attempts**: 3

## Functions
- **`login()`**: Prompts user to input password and checks it against the predefined password.
- **`loading_animation()`**: Displays a loading animation on the LCD.
- **`clear_line(int line)`**: Clears a specific line on the LCD.
- **`login_area()`**: Displays a message and waits for a key press after successful login.

## Usage
1. **Power up the system**.
2. **Enter the password** using the keypad.
3. **Observe the LCD** for login status and follow prompts.
4. After successful login, the system will display "LOGGING IN" and proceed to the next area.
5. If the password is incorrect after 3 attempts, the system will block further attempts until reset.

## Code

```cpp
#include <Keypad.h>
#include <LiquidCrystal.h>

const byte ROWS = 4; // Number of Keypad Rows
const byte COLS = 4; // Number of Keypad Columns
byte PIN_ROWS[] = { 2, 3, 4, 5 };
byte PIN_COLS[] = { 6, 7, 8, 9 };
char KEYS[4][4] = {
    { '1', '2', '3', 'A' },
    { '4', '5', '6', 'B' },
    { '7', '8', '9', 'C' },
    { '*', '0', '#', 'D' }
};

const int PIN_RS = A5;
const int PIN_E  = A4;
const int PIN_D4 = A3;
const int PIN_D5 = A2;
const int PIN_D6 = A1;
const int PIN_D7 = A0;
const int PIN_BUZZER = 11;
const int PASS_LENGTH = 5;
const char PASSWORD[] = { '1', '2', '3', 'A', 'B' };
const int MAX_ATTEMPT = 3;

Keypad keypad = Keypad( makeKeymap(KEYS), PIN_ROWS, PIN_COLS, ROWS, COLS );
LiquidCrystal lcd( PIN_RS, PIN_E, PIN_D4, PIN_D5, PIN_D6, PIN_D7 );

int attempt = 0;

void setup(){
    pinMode( PIN_BUZZER, OUTPUT );
    lcd.begin( 16, 2 );
}

void loop(){
    if( login() ) {
        attempt = 0;
        clear_line( 1 );
        lcd.print("LOGGING IN");
        loading_animation();
        login_area();
    } else {
        clear_line( 1 );
        if( ++attempt == MAX_ATTEMPT ) {
            lcd.print("-- BLOCKED --");
            while( true ); // Block program, reset Arduino to restart
        }
        String msg = "INVALID [" + String(attempt) + "/" + String(MAX_ATTEMPT) + "]";
        lcd.print(msg);
        delay( 1500 );
    }
}

void loading_animation() {
    for( int i = 0; i < 3; i++ ) {
        lcd.print(".");
        delay( 1000 );
    }
}

void clear_line( int line ) {
    lcd.setCursor( 0, line );
    lcd.print("                "); // Print 16 spaces to clear the line
    lcd.setCursor( 0, line );
}

void login_area() {
    lcd.clear();
    lcd.print("DuinoLab.com");
    lcd.setCursor(0,1);
    lcd.print("Press any key...");
    keypad.waitForKey();
    lcd.clear();
    lcd.print("LOGGING OUT");
    loading_animation();
}

bool login() {    
    boolean is_correct_password = true;
    char input;
    
    lcd.clear();
    lcd.print("> LOGIN AREA");
    lcd.setCursor( 0, 1 );
    lcd.print("INPUT:");
    
    for( int i = 0; i < PASS_LENGTH; i++ ) {
        input = keypad.waitForKey();
        if( input == '*' || input == '#' ) {
            i = -1;
            is_correct_password = true;
            clear_line( 1 );
            lcd.print("INPUT:");
        } else {
            if( is_correct_password ) {
                is_correct_password = (input == PASSWORD[i]);
            }
            lcd.print("*");
            digitalWrite( PIN_BUZZER, HIGH );
            delay( 200 );
            digitalWrite( PIN_BUZZER, LOW );
        }

    }

    return is_correct_password;
}
```

