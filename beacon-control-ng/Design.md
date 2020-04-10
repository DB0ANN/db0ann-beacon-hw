# Design of beacon control next generation


## Power Supply

Board is supplied by external PSU with -10 V, +5 V and +13.8 V. +13.8 V is
marked as +12 V in schematic.


### 3,3V Supply

A LDO is used to generate the 3.3 V Supply out of 5 V.

| Module    | I_avg mA | I_max mA | Comment         |
|-----------|----------|----------|-----------------|
| Lantronix |      110 |      270 | 10Base-T Idle   |
| µC        |        ? |       24 | 36 MHz from RAM |
| EEPROM    |    0.003 |        2 |                 |
|           |          |          |                 |
| Summ      |          |      296 |                 |

Power dissipation
P_max = I_max * ( U_in - U_out )
P_max = 296 mA * ( 5 V - 3.3 V )
P_max = 457 mW

The LDL1117S33R LDO is used. The LDL1117 variant should be preferred over the
LD1117 variant, because teh LDL1117 has a quiescent current of only 0.25 mA
compared to 5 mA of the LD1117.


## Digital IO filter

Zener Diode to limit Voltage to 3.3 Volts. Additionally all inputs connected to
5 V tolerant inputs.

Capacitor added for frequency filtering.

f_g = 1 / ( 2 * π * R * C )
f_g = 1 / ( 2 * π * 1k * 100n )
f_g = 1.59 kHz


## Analog IO

V_adc-max = 3.3 V


### Positive voltages

Used for OP Voltages, +13,8 V and +5 V.

Schematic:
    V+
    |
   .-.
10k| |
 R1| |
   '-'
    |      V_adc
    o----o----#---o
    |    |        |
   .-.   |       .-.
2k4| |  ---100n  | |50k
 R2| |  ---C1    | |Ri_ADC
   '-'   |       '-'
    |    |        |
   GND  GND      GND

V_adc = V_in * R2 || Ri_ADC / ( R1 + ( R2 || Ri_ADC ) )
V_adc = V_in * 2k29 / ( 10k + 2k29 )
V_adc = V_in * 0.186

V_in-max = V_adc-max / 0.186
V_in-max = 3.3 V / 0.186
V_in-max = 17.71 V


### Negative voltages

Used for -10 V.

Schematic:
    Up
    |
   .-.
10k| |
 R1| |
   '-'
    |              U_adc
    o------o-----o----#---o
    |      |     |        |
   .-.    .-.    |       .-.
10k| | 2k4| |   ---100n  | |50k
 R2| |  R3| |   ---C1    | |Ri_ADC
   '-'    '-'    |       '-'
    |      |     |        |
    Un    GND   GND      GND

R3 || Ri_ADC can be combined. For better readability in the following equations,
R3 is used instead of R3 || Ri_ADC.
Up positive voltage
Un negative voltage

1: Up = U1 + U3
2: UP = U1 + U2 + Un
3: I1 = I2 + I3

1: Up = I1*R1 + I3*R3
2: UP = I1*R1 + I2*R2 + Un
3: I1 = I2 + I3

=> R1 = R2
1: Up = I1*R1 + I3*R3
2: UP = I1*R1 + I2*R1 + Un
3: I1 = I2 + I3

=> eliminate I1 (3 in 1 and 3 in 2)
1: Up = I2*R1 + I3*R1 + I3*R3
2: UP = I2*R1 + I3*R1 + I2*R1 + Un

1: Up = I2*R1 + I3*(R1+R3)
2: UP = I2*2*R1 + I3*R1 + Un

1:   Up        R1+R3
I2 = -- - I3 * -----
     R1          R1

2:   Up-Un
I3 = ----- - I2*2
       R1

=> eliminate I2 (1 in 2)
     Up-Un         Up        R1+R3
I3 = ----- - 2 * [ -- - I3 * ----- ]
       R1          R1          R1

              R1+R3   Up+Un
I3 = I3 * 2 * ----- - -----
                R1      R1

              R1+R3   Up+Un
I3 = 2 * I3 * ----- - -----
                R1      R1

       R1+R3        Up+Un
I3 [ 2*----- -1 ] = -----
         R1           R1

     R1+2*R3   Up+Un
I3 * ------- = -----
        R1       R1

     Up + Un
I3 = -------
     R1+2*R3

                 R3
U3 = (Up+Un) * -------
               R1+2*R3

          R1+2*R3
Un = U3 * ------- - Up
            R3

=> R1 = 10k, R3 = 2k29
Un = U3 * 6.366 - Up


## CW filter

The CW signal filter was designed by DL8ZX. It was originally designed for 5 V
TTL input but we will generate only 3.3 V. To compensate this a gain of 1.5 is
added to the first OP stage.

Target gain
a = 5 V / 3.3 V = 1.515

    R1+R2   5k1+10k
a = ----- = ------- = 1.51
      R2      10k
