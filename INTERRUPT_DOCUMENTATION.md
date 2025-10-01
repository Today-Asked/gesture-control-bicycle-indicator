# Interrupt-Related Files and Code Sections Documentation

This document provides a comprehensive overview of all interrupt-related files and code sections in the BT_Client.X and BT_Server.X folders.

## Table of Contents
1. [BT_Client.X Interrupt Files](#bt_clientx-interrupt-files)
2. [BT_Server.X Interrupt Files](#bt_serverx-interrupt-files)
3. [Summary](#summary)

---

## BT_Client.X Interrupt Files

### 1. BT_Client.X/setting_hardaware/interrupt_manager.h
**Purpose:** Header file declaring the interrupt initialization function.

**Location:** `BT_Client.X/setting_hardaware/interrupt_manager.h`

**Code:**
```c
#ifndef _INTERRUPT_MANAGER_H
#define _INTERRUPT_MANAGER_H
void INTERRUPT_Initialize(void);

#endif
```

**Description:**
- Declares the `INTERRUPT_Initialize()` function
- Uses header guard to prevent multiple inclusions

---

### 2. BT_Client.X/setting_hardaware/interrupt_manager.c
**Purpose:** Implementation of interrupt initialization for the BT Client.

**Location:** `BT_Client.X/setting_hardaware/interrupt_manager.c`

**Code:**
```c
#include <xc.h>

void INTERRUPT_Initialize (void)
{
    RCONbits.IPEN = 1;      //enable Interrupt Priority mode
    INTCONbits.GIEH = 1;    //enable high priority interrupt
    INTCONbits.GIEL = 1;     //disable low priority interrupt

    //LED
    ADCON1 = 0x0f;          // Set ADCON1 register for digital mode
    LATA = 0x00;            // Clear LATA
    TRISAbits.TRISA2 = 0; 
    TRISAbits.TRISA5 = 0;   // Set RA1 as output
    TRISAbits.TRISA6 = 0;   // Set RA2 as output
    TRISAbits.TRISA7 = 0;   // Set RA3 as output
}
```

**Key Interrupt Configurations:**
- **Line 5:** `RCONbits.IPEN = 1` - Enables Interrupt Priority mode
- **Line 6:** `INTCONbits.GIEH = 1` - Enables high priority interrupts globally
- **Line 7:** `INTCONbits.GIEL = 1` - Enables low priority interrupts globally
- **Lines 9-15:** Configure LED pins (not directly interrupt-related but part of initialization)

---

### 3. BT_Client.X/main.c - Interrupt Service Routines
**Purpose:** Contains the actual interrupt service routines (ISRs) for handling interrupts.

**Location:** `BT_Client.X/main.c` (Lines 105-186)

#### High Priority ISR
**Code:**
```c
void __interrupt(high_priority) Hi_ISR(void)
{
    
}
```

**Description:**
- Empty high priority interrupt service routine
- Currently not handling any high priority interrupts
- Uses `__interrupt(high_priority)` compiler directive

#### Low Priority ISR
**Code:**
```c
// void interrupt low_priority Lo_ISR(void)
void __interrupt(low_priority)  Lo_ISR(void)
{
    if(RCIF)
    {
        if(RCSTAbits.OERR)
        {
            CREN = 0;
            Nop();
            CREN = 1;
        }
        
        LATAbits.LATA2 = 1;
        MyusartRead();
        char command[20];
        if(RCREG == '\r' || RCREG == '\n'){
            strcpy(command, GetString());
            ClearBuffer();
        }
        if(command[0] == 'L' && command[1] == 'R' && command[2] == 'N' && strlen(command) == 3){
            state = 1;
            ClearBuffer();
            strcpy(command, "");
            return;
        }
        else if(command[0] == 'L' && command[1] == 'R' && command[2] == 'F' && strlen(command) == 3){
            state = 2;
            ClearBuffer();
            strcpy(command, "");
            return;
        }
        else if(command[0] == 'L' && command[1] == 'L' && command[2] == 'N' && strlen(command) == 3){
            state = 3;
            ClearBuffer();
            strcpy(command, "");
            return;
        }
        else if(command[0] == 'L' && command[1] == 'L' && command[2] == 'F' && strlen(command) == 3){
            state = 4;
            ClearBuffer();
            strcpy(command, "");
            return;
        }
        else if(command[0] == 'B' && command[1] == 'N' && strlen(command) == 2){
            state = 5;
            ClearBuffer();
            strcpy(command, "");
            return;
        }
        else if(command[0] == 'B' && command[1] == 'F' && strlen(command) == 2){
            state = 6;
            ClearBuffer();
            strcpy(command, "");
            return;
        }
    }
    else if(TMR2IF){
        TMR2IF = 0;
        postpostscaler = (postpostscaler+1)%8;
        if(!postpostscaler){
            if(dir == -1){
                toggle_left();
            }else if(dir == 1){
                toggle_right();
            }else{}
        }
    }
    
   // process other interrupt sources here, if required

    return;
}
```

**Key Interrupt Handling:**
- **RCIF (Receive Interrupt Flag):** Handles UART receive interrupts
  - Checks for overrun errors (`RCSTAbits.OERR`)
  - Reads incoming UART data via `MyusartRead()`
  - Parses commands and updates system state:
    - `LRN` - Right light on (state = 1)
    - `LRF` - Right light off (state = 2)
    - `LLN` - Left light on (state = 3)
    - `LLF` - Left light off (state = 4)
    - `BN` - Brake on (state = 5)
    - `BF` - Brake off (state = 6)
- **TMR2IF (Timer 2 Interrupt Flag):** Handles timer interrupts
  - Used for LED blinking control
  - Toggles left/right indicators based on direction

---

### 4. BT_Client.X/setting_hardaware/uart.c - UART Interrupt Configuration
**Purpose:** Configures UART interrupts for serial communication.

**Location:** `BT_Client.X/setting_hardaware/uart.c` (Lines 32-39)

**Code:**
```c
void UART_Initialize() {
    // ... (other UART configuration)
    
    PIR1bits.TXIF = 0; //interrupt
    PIR1bits.RCIF = 1; //receiver changed
    TXSTAbits.TXEN = 1;           
    RCSTAbits.CREN = 1; //receiver    
    PIE1bits.TXIE = 0; //interrupt changed   
    IPR1bits.TXIP = 0;             
    PIE1bits.RCIE = 1; //interrupt         
    IPR1bits.RCIP = 0; //interrupt   
}
```

**Key Interrupt Configurations:**
- **Line 32:** `PIR1bits.TXIF = 0` - Clears UART transmit interrupt flag
- **Line 36:** `PIE1bits.TXIE = 0` - Disables UART transmit interrupt
- **Line 38:** `PIE1bits.RCIE = 1` - Enables UART receive interrupt
- **Line 39:** `IPR1bits.RCIP = 0` - Sets UART receive interrupt to low priority

---

### 5. BT_Client.X/setting_hardaware/setting.h
**Purpose:** Includes interrupt manager header.

**Location:** `BT_Client.X/setting_hardaware/setting.h` (Line 11)

**Code:**
```c
#include "interrupt_manager.h"
```

---

## BT_Server.X Interrupt Files

### 1. BT_Server.X/setting_hardaware/interrupt_manager.h
**Purpose:** Header file declaring the interrupt initialization function.

**Location:** `BT_Server.X/setting_hardaware/interrupt_manager.h`

**Code:**
```c
#ifndef _INTERRUPT_MANAGER_H
#define _INTERRUPT_MANAGER_H
void INTERRUPT_Initialize(void);

#endif
```

**Description:**
- Identical to BT_Client.X version
- Declares the `INTERRUPT_Initialize()` function
- Uses header guard to prevent multiple inclusions

---

### 2. BT_Server.X/setting_hardaware/interrupt_manager.c
**Purpose:** Implementation of interrupt initialization for the BT Server.

**Location:** `BT_Server.X/setting_hardaware/interrupt_manager.c`

**Code:**
```c
#include <xc.h>

void INTERRUPT_Initialize (void)
{
    RCONbits.IPEN = 1;      //enable Interrupt Priority mode
    INTCONbits.GIEH = 1;    //enable high priority interrupt
    INTCONbits.GIEL = 1;     //disable low priority interrupt

    //LED
    ADCON1 = 0x0f;          // Set ADCON1 register for digital mode
    LATA = 0x00;            // Clear LATA
    TRISA = 0x00;
    TRISAbits.TRISA0 = 1;   // set RA0 as input
    
    //BUTTOM
    TRISB = 0x01;
    LATBbits.LATB0 = 0;
    
    INTCONbits.INT0IE = 0;
    INTCONbits.INT0IF = 0;
}
```

**Key Interrupt Configurations:**
- **Line 5:** `RCONbits.IPEN = 1` - Enables Interrupt Priority mode
- **Line 6:** `INTCONbits.GIEH = 1` - Enables high priority interrupts globally
- **Line 7:** `INTCONbits.GIEL = 1` - Enables low priority interrupts globally
- **Line 19:** `INTCONbits.INT0IE = 0` - Disables external interrupt 0
- **Line 20:** `INTCONbits.INT0IF = 0` - Clears external interrupt 0 flag
- **Lines 9-17:** Configure LED and button pins

**Differences from BT_Client.X:**
- Includes button configuration (TRISB)
- Configures external interrupt INT0 (disabled)
- Different LED pin configurations

---

### 3. BT_Server.X/main.c
**Purpose:** Main application file for BT Server.

**Location:** `BT_Server.X/main.c`

**Note:** The BT_Server.X/main.c file does NOT contain explicit interrupt service routines (ISRs). The server relies on:
- Interrupt initialization from `interrupt_manager.c`
- UART interrupt configuration from `uart.c`
- Polling-based operation in the main loop

**Main Loop:**
```c
void main(void) 
{
    SYSTEM_Initialize() ;
    
    int blinker_dir = 0;
    int prev_bend_sensor_val;
    
    while(1) {
        Check_ADC(prev_bend_sensor_val);
        Check_Gyroscope(blinker_dir);
        __delay_ms(20);
    }
    return;
}
```

The server uses polling in the main loop rather than interrupt-driven architecture for sensor checking.

---

### 4. BT_Server.X/setting_hardaware/uart.c - UART Interrupt Configuration
**Purpose:** Configures UART interrupts for serial communication.

**Location:** `BT_Server.X/setting_hardaware/uart.c` (Lines 32-39)

**Code:**
```c
void UART_Initialize() {
    // ... (other UART configuration)
    
    PIR1bits.TXIF = 0; //interrupt
    PIR1bits.RCIF = 1; //receiver changed
    TXSTAbits.TXEN = 1;           
    RCSTAbits.CREN = 1; //receiver    
    PIE1bits.TXIE = 0; //interrupt changed   
    IPR1bits.TXIP = 0;             
    PIE1bits.RCIE = 1; //interrupt         
    IPR1bits.RCIP = 0; //interrupt   
}
```

**Key Interrupt Configurations:**
- **Line 32:** `PIR1bits.TXIF = 0` - Clears UART transmit interrupt flag
- **Line 36:** `PIE1bits.TXIE = 0` - Disables UART transmit interrupt
- **Line 38:** `PIE1bits.RCIE = 1` - Enables UART receive interrupt
- **Line 39:** `IPR1bits.RCIP = 0` - Sets UART receive interrupt to low priority

**Note:** Identical to BT_Client.X UART interrupt configuration, with the only difference being the baud rate comment (38400 vs no comment).

---

### 5. BT_Server.X/setting_hardaware/setting.h
**Purpose:** Includes interrupt manager header.

**Location:** `BT_Server.X/setting_hardaware/setting.h` (Line 11)

**Code:**
```c
#include "interrupt_manager.h"
```

---

## Summary

### BT_Client.X Interrupt Architecture
The BT_Client.X project uses a **interrupt-driven architecture** for UART communication and timer-based LED control:

1. **Interrupt Initialization:** Configured in `interrupt_manager.c`
2. **High Priority ISR:** Empty, reserved for future use
3. **Low Priority ISR:** Handles:
   - UART receive interrupts (RCIF) - Processes Bluetooth commands
   - Timer 2 interrupts (TMR2IF) - Controls LED blinking patterns
4. **UART Interrupts:** Receive interrupt enabled, transmit interrupt disabled

### BT_Server.X Interrupt Architecture
The BT_Server.X project uses a **hybrid architecture** with interrupt initialization but polling-based main operation:

1. **Interrupt Initialization:** Configured in `interrupt_manager.c`
   - Includes external interrupt INT0 configuration (disabled)
   - Includes button input configuration
2. **No ISRs in main.c:** Server uses polling instead of interrupt-driven event handling
3. **UART Interrupts:** Configured similarly to client but may be unused
4. **Main Loop:** Uses polling with `__delay_ms(20)` to check sensors periodically

### Key Differences Between BT_Client.X and BT_Server.X

| Aspect | BT_Client.X | BT_Server.X |
|--------|-------------|-------------|
| **ISR Implementation** | Yes (Hi_ISR and Lo_ISR) | No explicit ISRs |
| **Operation Mode** | Interrupt-driven | Polling-based |
| **UART Interrupt Handling** | Active (processes commands in ISR) | Configured but not actively used |
| **Timer Interrupts** | Yes (TMR2 for LED blinking) | Not used |
| **External Interrupts** | Not configured | INT0 configured but disabled |
| **Button Input** | Not configured | Configured in interrupt_manager.c |

### Interrupt Registers Used

Common registers used across both projects:
- **RCONbits.IPEN:** Interrupt Priority Enable bit
- **INTCONbits.GIEH:** Global Interrupt Enable (High priority)
- **INTCONbits.GIEL:** Global Interrupt Enable (Low priority)
- **PIE1bits.RCIE:** UART Receive Interrupt Enable
- **PIR1bits.RCIF:** UART Receive Interrupt Flag
- **IPR1bits.RCIP:** UART Receive Interrupt Priority

Additional registers in BT_Client.X:
- **TMR2IF:** Timer 2 Interrupt Flag

Additional registers in BT_Server.X:
- **INTCONbits.INT0IE:** External Interrupt 0 Enable
- **INTCONbits.INT0IF:** External Interrupt 0 Flag

### Files Summary

**BT_Client.X Interrupt-Related Files:**
1. `BT_Client.X/setting_hardaware/interrupt_manager.h` - Header declaration
2. `BT_Client.X/setting_hardaware/interrupt_manager.c` - Interrupt initialization
3. `BT_Client.X/main.c` - ISR implementations (Lines 105-186)
4. `BT_Client.X/setting_hardaware/uart.c` - UART interrupt configuration (Lines 32-39)
5. `BT_Client.X/setting_hardaware/setting.h` - Include statement (Line 11)

**BT_Server.X Interrupt-Related Files:**
1. `BT_Server.X/setting_hardaware/interrupt_manager.h` - Header declaration
2. `BT_Server.X/setting_hardaware/interrupt_manager.c` - Interrupt initialization
3. `BT_Server.X/main.c` - Main loop (no ISRs, polling-based)
4. `BT_Server.X/setting_hardaware/uart.c` - UART interrupt configuration (Lines 32-39)
5. `BT_Server.X/setting_hardaware/setting.h` - Include statement (Line 11)

---

## Additional Notes

### Build Artifacts
The repository also contains compiled/preprocessed files in build directories:
- `BT_Client.X/build/default/production/main.i`
- `BT_Client.X/build/default/debug/main.i`
- `BT_Server.X/build/default/debug/setting_hardaware/interrupt_manager.i`

These are generated files containing assembly language and preprocessor output, not source code files.

### Microcontroller
Based on the register names and XC8 compiler includes, this project appears to be targeting a **PIC18F4520** microcontroller.

---

*Document generated to fulfill the requirement: "find files and code sections regarding interrupt in the folder BT_Client.X or BT_Server.X"*
