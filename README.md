# Sensorbox-2
Sensorbox2 measures the current(A) and current direction for up to three phases with CT's and a MAINS voltage input.
It can also receive these measurements directly from a (D)SMR5 smart meters P1 port.
These measurements are then sent every two seconds to the connected SmartEVSE(s).

The Sensorbox2 uses a PIC microcontroller which currently does all the processing. It takes the measurements from the CT inputs and P1 port and can send the data over the RS485 bus.
It will be programmed initially by the ESP32 module. At powerup the ESP32 looks for a Sensorbox2.hex file in the /data folder and, when found programs the PIC18F26K40 microcontroller.

Please note that this also means that the PIC has control over the RS485 bus. If you want full controll, keep the PIC in reset by holding the PIC reset line low.

I'll release a version of the PIC firmware soon that will only do the CT measurements, and will not use the RS485 bus, nor use the P1 port.
On this version the ESP32 will handle everything else.

## Modbus Registers

Modbus RTU address is fixed to 0x0A, speed is 9600 bps 

   
Input Registers (FC=04)(Read Only):
  
 

    Register  Register  
     Address   length (16 bits)
     0x0000      1       Sensorbox version 2             = 0x0014 = 4 wire, CW rotation
                                                           0x0015 = 4 wire, CCW rotation
                                                           0x0016 = 3 wire, CW rotation      
                                                           0x0017 = 3 wire, CCW rotation  
     0x0001      1       DSMR Version(MSB),CT's or P1(LSB) 0x3283 = DSMR version 50, P1 port connected (0x80) CT's Used (0x03)                    
     0x0002      2       Volts L1 (32 bit floating point), Smartmeter P1 data
     0x0004      2       Volts L2 (32 bit floating point), Smartmeter P1 data
     0x0006      2       Volts L3 (32 bit floating point), Smartmeter P1 data
     0x0008      2       Amps L1 (32 bit floating point), Smartmeter P1 data
     0x000A      2       Amps L2 (32 bit floating point), Smartmeter P1 data
     0x000C      2       Amps L3 (32 bit floating point), Smartmeter P1 data
     0x000E      2       Amps L1 (32 bit floating point), CT imput 1
     0x0010      2       Amps L2 (32 bit floating point), CT input 2
     0x0012      2       Amps L3 (32 bit floating point), CT input 3
    
    Holding Registers (FC=06)(Write):
    
     Register  Register  
     Address   length (16 bits) 
     0x0800      1       Field rotation setting (bit 0)      00000000 0000000x -> 0= Rotation right 1= Rotation Left
                         3/4 wire configuration (bit 1)      00000000 000000x0 -> 0= 4Wire, 1= 3Wire

## Example:

    Request Sensorbox2 data:
    0a 04 00 00 00 14 f1 7e
    
    Sensorbox2 replies with:
    0a 04 28 00 14 32 83 43 66 80 00 43 66 4c cd 43 69 19 9a 3e 12 9a 61 3f 9f 83 80 be 41 4a 5a 3b ea 00 3b bb 6f f3 6b 3a 45 8d e3 ca cb
    4 wire CW --^^
    DSMR ver(50) --^^
    P1 + CT connected ^^
    Volts L1 ------------| 230.5 V  |			 
    					 
    Request 3 wire grid setting:
    0a 06 08 00 00 02 0b 10		
    
    Next sensorbox2 data:
    0a 04 28 00 16 32 03 43 66 e6 66 43 66 80 00 43 69 19 9a 3d 66 9c 53 3f 97 99 d2 be a2 8a 28 bb 59 73 10 bb 08 29 4d 3b 8f 23 79 ca e6 			 
    ------------^^ changed to 3 wire, CW setting.






