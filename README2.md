******************************************************************************
;   This file is a basic template for creating relocatable assembly code for  *
;   a PIC18F1330. Copy this file into your project directory and modify or    *
;   add to it as needed.                                                      *
;                                                                             *
;   The PIC18FXXXX architecture allows two interrupt configurations. This     *
;   template code is written for priority interrupt levels and the IPEN bit   *
;   in the RCON register must be set to enable priority levels. If IPEN is    *
;   left in its default zero state, only the interrupt vector at 0x008 will   *
;   be used and the WREG_TEMP, BSR_TEMP and STATUS_TEMP variables will not    *
;   be needed.                                                                *
;                                                                             *
;   Refer to the MPASM User's Guide for additional information on the         *
;   features of the assembler and linker.                                     *
;                                                                             *
;******************************************************************************
;                                                                             *
;    Filename:                                                                *
;    Date:                                                                    *
;    File Version:                                                            *
;                                                                             *
;    Author:                                                                  *
;    Company:                                                                 *
;                                                                             * 
;******************************************************************************
;                                                                             *
;    Files required: P18F1330.INC                                             *
;                                                                             *
;******************************************************************************

	LIST P=18F1330, F=INHX32 ;directive to define processor and file format
	#include <P18F1330.INC>	 ;processor specific variable definitions

;******************************************************************************
;Configuration bits
;Microchip has changed the format for defining the configuration bits, please 
;see the .inc file for futher details on notation.  Below are a few examples.



;   Oscillator Selection:
;    CONFIG	OSC = LP             ;LP
     CONFIG  OSC = INTIO1
     CONFIG  WDT = OFF



;******************************************************************************
;Variable definitions
; These variables are only needed if low priority interrupts are used. 
; More variables may be needed to store other special function registers used
; in the interrupt routines.

		UDATA

WREG_TEMP	RES	1	;variable in RAM for context saving 
STATUS_TEMP	RES	1	;variable in RAM for context saving
BSR_TEMP	RES	1	;variable in RAM for context saving

		UDATA_ACS

EXAMPLE		RES	1	;example of a variable in access RAM

;******************************************************************************
;EEPROM data
; Data to be programmed into the Data EEPROM is defined here

DATA_EEPROM	CODE	0xf00000

		DE	"Test Data",0,1,2,3,4,5

;******************************************************************************
;Reset vector
; This code will start executing when a reset occurs.

RESET_VECTOR	CODE	0x0000

		goto	Main		;go to start of main code

;******************************************************************************
;High priority interrupt vector
; This code will start executing when a high priority interrupt occurs or
; when any interrupt occurs if interrupt priorities are not enabled.

HI_INT_VECTOR	CODE	0x0008

		bra	HighInt		;go to high priority interrupt routine

;******************************************************************************
;Low priority interrupt vector
; This code will start executing when a low priority interrupt occurs.
; This code can be removed if low priority interrupts are not used.

LOW_INT_VECTOR	CODE	0x0018

		bra	LowInt		;go to low priority interrupt routine

;******************************************************************************
;High priority interrupt routine
; The high priority interrupt code is placed here.

		CODE

HighInt:

;	*** high priority interrupt code goes here ***


		retfie	FAST

;******************************************************************************
;Low priority interrupt routine
; The low priority interrupt code is placed here.
; This code can be removed if low priority interrupts are not used.

LowInt:
		movff	STATUS,STATUS_TEMP	;save STATUS register
		movff	WREG,WREG_TEMP		;save working register
		movff	BSR,BSR_TEMP		;save BSR register

;	*** low priority interrupt code goes here ***


		movff	BSR_TEMP,BSR		;restore BSR register
		movff	WREG_TEMP,WREG		;restore working register
		movff	STATUS_TEMP,STATUS	;restore STATUS register
		retfie

;******************************************************************************
;Start of main program
; The main program code is placed here.

Main:

    Main:

    BSF OSCCON,  6  ;Oscillator set to 8 MHz
    BSF OSCCON,  5
    BSF OSCCON,  4



    BCF T0CON, 5    ;Init TMR0 as a timer (not an event counter)

    BCF T0CON, 6    ;Init TMR0 as a 16-bit counter (not 8 bit)

    BCF T0CON, 3    ;Init TMR0 PreScaler ON

;prescale is 256----------------------------
;prescale1:
     BSF T0CON, 2    ;Prescaler 1:256
     BSF T0CON, 1    ;Prescaler 1:256
     BSF T0CON, 0    ;Prescaler 1:256

    BCF T0CON, 7    ;Init Timer to be OFF
    BSF TRISB, 4     ;Input switch
    BCF TRISB, 0     ;Green
    BCF TRISB, 1     ;Red

    ;BSF PORTB, 4    ;Init PORTB Bit 0 Low

led:
    BCF PORTB,0
    BCF PORTB,1
    BTFSC PORTB,4
    BRA StartX
    BRA led
;---------------------------------------------------------------Level 1

;-------------------------------------led1


StartX:
    BSF PORTB,0
    MOVLW   h'A4'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'70'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N
WaitMore:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore
    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,0

    
 ;------------------------------------led2

StartX2:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore2:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore2

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led

       
    ;------------------------------------led3

StartX3:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore3:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore3

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led
    
    
        
    ;-------------------------------led4

StartX4:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore4:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore4

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led

;------------------------------------------------------------------Level 2 

;-------------------------------------led1


StartX21:
    BSF PORTB,0
    MOVLW   h'A4'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'70'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N
WaitMore21:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore21
    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,0

    
 ;------------------------------------led2

StartX22:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore22:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore22 

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led

       
    ;------------------------------------led3

StartX31:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore32:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore32

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led
    
    
        
    ;-------------------------------led4

StartX42:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore42:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore42

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led
;------------------------------------------------------------------Level 3

;-------------------------------------led1


StartX33:
    BSF PORTB,0
    MOVLW   h'A4'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'70'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N
Waitmore33:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore33
    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,0

    
 ;------------------------------------led2

StartX23:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore23:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore23

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led

       
    ;------------------------------------led3

StartX3:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore33:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore33

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led
    
    
        
    ;-------------------------------led4

StartX43:
    BCF PORTB,0
    BSF PORTB,1
    MOVLW   h'67'              ;Starting Count = 0000
    MOVWF   TMR0H
    MOVLW   h'60'
    MOVWF   TMR0L
    BSF     T0CON, 7        ;Turn Timer 0N

WaitMore43:
    BTFSS   INTCON, TMR0IF  ;Check for Count = FFFF
    BRA     WaitMore43

    BCF     T0CON, 7        ;Turn Timer OFF
    BCF     INTCON, TMR0IF  ;Reset TIMER interrupt low
    BCF     PORTB,1
    BRA     led
;End of program

		END
  ;******************************************************************************
