# AS5600
AS560 Micropython library for reading this magnetic sensor


AS5600.py

**What is an  AS5600?**

From the data sheet https://ams.com/documents/20143/36005/AS5600_DS000365_5-00.pdf

### `12-Bit Programmable Contactless Potentiometer`

`The AS5600 is an easy to program magnetic rotary position sensor with a high-resolution 12-bit analog or PWM output. This contactless system measures the absolute angle of a diametric magnetized on-axis magnet. "`

This is a Micropython library for reading an AS5600 using I2C.  It was developed on a Seeedstudio board and Raspberry Pi Pico.  It should work with other MCU's running Micropython but I2C setup may differ on on other boards.  It may work on circuit python but I have not tested this.

The field names are taken from the data sheet. All registers in the data sheet are implemented, as simple attributes of the AS5600 class.   You will need to read the datasheet to use the library.

### HOW IT WORKS (Programming)

**NB: YOU WILL STILL HAVE TO READ THE DATASHEET**

1. Each register or bitfield named in the datasheet (capitalised) is made into an attribute of the AS5600_low class (see Data descriptor below)
2. A higher level class AS5600 inherits from AS5600_low.  This allows friendlier names etc, without polluting the basics.  (Note: I dont think you can make friendler attribute names at this level unless you make a new descriptor).
3. You have to call super() in __init__ in the higher level class
4. Micropython does not seem to have __attribute__ etc so I use Data descriptors which a probably better and simpler anyway.  The actual I2C calls are done in the data descriptor class.
5. Configuration attributes are writeable and/or burnable but do not change in normal useage.  Therefore they are cached, on first read.  The cache is updated if the register is written to.  The CONFIG register in particular has lots of little bit fields so you do not want to have to read over I2C for every little bitfield.

From the code:

```
   #Use descriptors to read and write a bit field from a register
    #1. we read one or two bytes from i2c
    #2. We shift the value so that the least significant bit is bit zero
    #3. We mask off the bits required  (most values are 12 bits hence m12)
    ZMCO=      RegDescriptor(r.ZMCO,shift=0,mask=3,buffsize=1) #1 bit
    ZPOS=      RegDescriptor(r.ZPOS,0,m12) #zero position
    MPOS=      RegDescriptor(r.MPOS,0,m12) #maximum position
    MANG=      RegDescriptor(r.MANG,0,m12) #maximum angle (alternative to above)
    CONF=      RegDescriptor(r.CONF,0,(1<<14)-1) # this register has 14 bits (see below)
    RAWANGLE=  RegDescriptor(r.RAWANGLE,0,m12) 
    ANGLE   =  RegDescriptor(r.ANGLE,0,m12) #angle with various adjustments (see datasheet)
    STATUS=    RegDescriptor(r.STATUS,0,m12) #basically strength of magnet
    AGC=       RegDescriptor(r.AGC,0,0xF,1) #automatic gain control
    MAGNITUDE= RegDescriptor(r.MAGNITUDE,0,m12) #? something to do with the CORDIC for atan RTFM
    BURN=      RegDescriptor(r.BURN,0,0xF,1)

    #Configuration bit fields
    PM =      RegDescriptor(r.CONF,0,0x3) #2bits Power mode
    HYST =    RegDescriptor(r.CONF,2,0x3) # hysteresis for smoothing out zero crossing
    OUTS =    RegDescriptor(r.CONF,4,0x3) # HARDWARE output stage ie analog (low,high)  or PWM
    PWMF =    RegDescriptor(r.CONF,6,0x3) #pwm frequency
    SF =      RegDescriptor(r.CONF,8,0x3) #slow filter (?filters glitches harder) RTFM
    FTH =     RegDescriptor(r.CONF,10,0x7) #3 bits fast filter threshold. RTFM
    WD =      RegDescriptor(r.CONF,13,0x1) #1 bit watch dog - Kicks into low power mode if nothing changes
    
    #status bit fields. ?having problems getting these to make sense
    MH =      RegDescriptor(r.STATUS,3,0x1) #2bits  Magnet too strong (high)
    ML =      RegDescriptor(r.STATUS,4,0x1) #2bits  Magnet too weak (low)
    MD =      RegDescriptor(r.STATUS,5,0x1) #2bits  Magnet detected
    
```



#### HOW IT WORKS (HARDWARE).

*Still testing.*

#### Example file.

This is an early work in progress.  I ran a small  endless loop program then waved a magnet around.  I seemed to get sensible values from RAWANGLE but obviously a mechanical better setup is required.

The MD field never seemed to say a magnet was not detected although reasonable values were being generated by RAW ANGLE.  I am not sure why this is so, at this stage.  So some debugging will be required.  

AGC levels seem very low.
