This repository contains the code for a crude oil heater simulator developed using assembly language for the PIC16F877A microcontroller. The simulation controls various inputs and outputs related to a crude oil heating system, ensuring the proper functioning of burners, pumps, valves, and safety mechanisms.

Features

Input Sensors:
Flame Detectors (4 channels)
Temperature Sensor (TT_101)
Pressure Sensors (PT_101, PT_102, PT_103)
Tank Level Sensors (T_201_NA, T_201_NB, T_202_NA, T_202_NB)
Gas Line Presence Sensor

Output Controls:
Ignition Sparkers (4 channels)
Line Pumps (3 channels)
Flow Control Valves (3 channels)
Burners (4 channels)
Solenoid Valves (4 channels)
Vent Controller

Operation

Initialization:
Configures input and output ports.
Clears all output registers.

Main Process:

Continuously monitors input conditions.
Controls the activation of pumps, burners, and valves based on sensor readings.
Ensures the safety protocols are followed, halting operations if unsafe conditions are detected.

Safety Procedures:
Monitors various burner combinations to prevent unsafe operations.
Emergency shutdown procedure (PARO_ACCION) engages when unsafe conditions are detected.

Subroutines:
VACIADO1: Handles tank emptying process.
SIN_GAS: Pauses operation if gas is absent.
T_20MIN: Implements a 20-minute delay using nested loops.
NIVEL: Manages tank level adjustments.
PRUEBA: Conducts a test cycle to ensure system readiness.

ASSEMBLER CODE: 

;Configuración inicial del PIC
#include <p16f877a.inc>

;Definición de las variables

;ENTRADAS AL PROCESO

;PUERTO A

DT_LLAMA2    EQU       0       ;DETENTOR DE LLAMA 2
DT_LLAMA3    EQU       1       ;DETENTOR DE LLAMA 3
DT_LLAMA4    EQU       2       ;DETENTOR DE LLAMA 4

;SALIDAS DEL PROCESO

;PUERTO A

CHISPA2      EQU       3       ;CHISPA DE PILOTO 2
CHISPA4      EQU       5       ;CHISPA DE PILOTO 4

;ENTRADAS DEL PROCESO

;PUERTO B

TT_101       EQU       0       ;SENSOR DE TEMPERATURA
PT_101       EQU       1       ;SENSOR DE PRESION 1
PT_102       EQU       2       ;SENSOR DE PRESION 2
PT_103       EQU       3       ;SENSOR DE PRESION 3
T_201_NA     EQU       4       ;SENSOR DE NIVEL ALTO DEL TANQUE 1
T_201_NB     EQU       5       ;SENSOR DE NIVEL BAJO DEL TANQUE 1
T_202_NA     EQU       6       ;SENSOR DE NIVEL ALTO DEL TANQUE 2
T_202_NB     EQU       7       ;SENSOR DE NIVEL ALTO DEL TANQUE 2

;PUERTO E

LINEAGAS     EQU       0       ;SENSOR DE PRESENCIA DE GAS EN LA LINEA DE QUEMADORES O PILOTOS
DT_LLAMA1    EQU       1       ;DETENTOR DE LLAMA 1

;SALIDA DEL PROCESO
;PUERTO E

CHISPA3      EQU       2       ;CHISPA DE PILOTO 3 

;SALIDAS DEL PROCESO

;PUERTO C

BOMBA1       EQU       0       ;BOMBA DE LINEA DE TANQUE 1
BOMBA2       EQU       1       ;BOMBA DE LINEA DE TANQUE 2
BOMBA3       EQU       2       ;BOMBA DE TANQUE 2
FCV_101      EQU       3       ;VALVULA DE LINEA DE TANQUE 1
FCV_102      EQU       4       ;VALVULA DE LINEA DE QUEMADORES
FCV_103      EQU       5       ;VALVULA DE LINEA DE TANQUE 2
FC           EQU       6       ;CONTROLADOR DE VENTEO
CHISPA1      EQU       7       ;CHISPA DE PILOTO 1

;PUERTO D

SV_101       EQU       0       ;VALVULA SELENOIDE 1
SV_102       EQU       1       ;VALVULA SELENOIDE 2
SV_103       EQU       2       ;VALVULA SELENOIDE 3
SV_104       EQU       3       ;VALVULA SELENOIDE 4
QUEMADOR1    EQU       4       ;QUEMADOR 1
QUEMADOR2    EQU       5       ;QUEMADOR 2
QUEMADOR3    EQU       6       ;QUEMADOR 3
QUEMADOR4    EQU       7       ;QUEMADOR 4

;REGISTRO DE PROPOSITO GENERAL

T1           EQU       0X36    ;VARIABLE DE TIEMPÒ 1
T2           EQU       0X37    ;VARIABLE DE TIEMPO 2
T3           EQU       0X38    ;VARIABLE DE TIEMPO 3

;Definición de los registros

STATUS       EQU       0x03
TRISA        EQU       0x85
PORTA        EQU       0x05
TRISB        EQU       0x86
PORTB        EQU       0x06

;Inicialización

ORG 0x00
    GOTO      PRINCIPAL

PRINCIPAL
;Configuración de puertos

    BSF       STATUS, RP0      ;COLOCANDO EN 1 EL BIT 5 (RP0) DEL REGISTRO STATUS
    BCF       STATUS, RP1      ;COLOCANDO EN 0 EL BIT 6 (RP1) DEL REGISTRO STATUS
    MOVLW     b'11111111'      ;DECLARANDO LOS 8 BITS COMO ENTRADAS 
    MOVWF     TRISB
    MOVLW     .6
    MOVWF     ADCON1
    MOVLW     b'00000011'           ;DECLARANDO LOS 3 BITS COMO ENTRADAS
    MOVWF     TRISE
    CLRF      TRISC            ;PUERTO C CONFIGURADO COMO SALIDA
    CLRF      TRISD            ;PUERTO D CONFIGURADO COMO SALIDA 
    MOVLW     b'000111'
    MOVWF     TRISA            ;PRIMEROS TRES PINES DEL PUERTO A COMO ENTRADA Y LOS ULTIMOS TRES COMO SALIDA 
    BCF       STATUS, RP0      ;COLOCANDO EN 0 EL BIT 5 (RP0) DEL REGISTRO STATUS
     
;Inicialización de variables

    CLRF      PORTB
    CLRF      PORTC
    CLRF      PORTD
    CLRF      PORTE

;Verificación de condiciones iniciales
 
INICIO
   
    BTFSS    PORTB, TT_101
    GOTO     $-1
    BTFSS    PORTB, PT_103
    GOTO     $-1
    BTFSS    PORTB, PT_101
    GOTO     $-1
    BTFSC    PORTB, T_202_NB
    CALL     VACIADO1
    BTFSS    PORTB, T_201_NA
    GOTO     $-1
    BTFSC    PORTE, LINEAGAS
    CALL     SIN_GAS
    BTFSC    PORTE, DT_LLAMA1
    GOTO     $-1
    BSF      PORTC, FC
    
;PROCESO PRINCIPAL

    CLRF     PORTD
    BSF      PORTC, FCV_102
    CALL     PRUEBA
    BTFSS    PORTB, PT_102
    GOTO     $-1
    BSF      PORTD, SV_101
    BSF      PORTC, CHISPA1
    BTFSS    PORTE, DT_LLAMA1
    GOTO     $-1 
    BSF      PORTD,QUEMADOR1
    
    BSF      PORTD, SV_102
    BSF      PORTA, CHISPA2
    BTFSS    PORTA, DT_LLAMA2
    GOTO     $-1 
    BSF      PORTD, QUEMADOR2
    
   
    BSF      PORTD, SV_103
    BSF      PORTE, CHISPA3
    BTFSS    PORTA, DT_LLAMA3
    GOTO     $-1 
    BSF      PORTD, QUEMADOR3
    
    BSF      PORTD, SV_104
    BSF      PORTA, CHISPA4
    BTFSS    PORTA,DT_LLAMA4
    GOTO     $-1 
    BSF      PORTD,QUEMADOR4
    
    BTFSS    PORTB,T_201_NA
    GOTO     $-1
    BSF      PORTC, BOMBA1
    BSF      PORTC, FCV_101
    BTFSS    PORTB, PT_101
    GOTO     $-1
    BSF      PORTC, FC
    BSF      PORTC, FCV_103
    BTFSS    PORTB, TT_101
    GOTO     $-1
    CALL     T_20MIN
    BSF      PORTC, BOMBA2
    BTFSS    PORTB,PT_103
    GOTO     $-1
    BTFSS    PORTB, T_202_NA
    GOTO     $-1
    BSF      PORTC, BOMBA3
    
;CONDICIONES DE SEGURIDAD

;CASO1   
    BTFSS    PORTD, QUEMADOR1
    GOTO     $+1
    BTFSS    PORTD, QUEMADOR2
    GOTO     PARO_ACCION
;CASO2   
    BTFSS    PORTD, QUEMADOR1
    GOTO     $+1
    BTFSS    PORTD, QUEMADOR3
    GOTO     PARO_ACCION
;CASO3
    BTFSS    PORTD, QUEMADOR1
    GOTO     $+1
    BTFSS    PORTD, QUEMADOR4
    GOTO     PARO_ACCION
;CASO4
    BTFSS    PORTD, QUEMADOR2
    GOTO     $+1
    BTFSS    PORTD, QUEMADOR3
    GOTO     PARO_ACCION
;CASO5
    BTFSS    PORTD, QUEMADOR2
    GOTO     $+1
    BTFSS    PORTD, QUEMADOR4
    GOTO     PARO_ACCION
;CASO6
    BTFSS    PORTD, QUEMADOR3
    GOTO     $+1
    BTFSS    PORTD, QUEMADOR4
    GOTO     PARO_ACCION
   
    BTFSS    PORTB, T_201_NB
    GOTO     PARO_ACCION
    
    BTFSS    PORTC, BOMBA1
    GOTO     PARO_ACCION
    
    BTFSS    PORTC, BOMBA2
    GOTO     PARO_ACCION
    
    GOTO     INICIO

;PROCESO DE PARO DE EMERGENCIA

PARO_ACCION
    BCF      PORTC, FCV_102
    BCF      PORTC, CHISPA1
    BTFSC    PORTE, DT_LLAMA1
    GOTO     $+1
    BCF      PORTD, QUEMADOR1
    BCF      PORTD, QUEMADOR2
    BCF      PORTD, QUEMADOR3
    BCF      PORTD, QUEMADOR4
    BSF      PORTC, FC
    BCF      PORTC, BOMBA1
    BCF      PORTC, FCV_101
    BCF      PORTC, BOMBA2
    BCF      PORTC, FCV_103
    BSF      PORTC, BOMBA3
    CALL     T_20MIN
    BCF      PORTC, BOMBA3
    BTFSC    PORTB, T_202_NA
    CALL     NIVEL
    GOTO     INICIO 

;SUBRUTINAS    
    
VACIADO1      BSF       PORTC, BOMBA3
              BTFSC     PORTB, T_202_NB
	      GOTO      $-1
	      BCF       PORTC, BOMBA3
	      RETURN
	    
SIN_GAS       BCF       PORTC, FCV_102
              BTFSC     PORTE, LINEAGAS
	      GOTO      $-1
	      RETURN
	      
T_20MIN       MOVLW     .25
              MOVWF     T3 
X3	      CALL      RETARDO1
	      DECFSZ    T3
	      GOTO      X3
	      RETURN
	  
RETARDO1      MOVLW     .100  
	      MOVWF     T2
X2	      CALL      RETARDO2
              DECFSZ    T2
	      GOTO      X2
	      RETURN
	      
RETARDO2      MOVLW     .249
              MOVWF     T1
X1	      NOP
	      DECFSZ    T1
	      GOTO      X1
	      RETURN
	      
NIVEL         BSF       PORTC, BOMBA3
              BTFSC     PORTB, T_202_NA
	      GOTO      $-1
	      BCF       PORTC, BOMBA3
	      RETURN
	     
PRUEBA        BTFSC    PORTB, T_201_NA
              GOTO     $-1
	      BCF      PORTC, FC
	      BCF      PORTC, FCV_102
	      BTFSS    PORTB, T_201_NA
	      GOTO     $-1
	      BSF      PORTC, FC
	      BSF      PORTC, FCV_102
	      BTFSC    PORTB, T_201_NA
	      GOTO     $-1
	      RETURN
	      
	      
              END

