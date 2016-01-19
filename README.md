# MSSP - Master Slave Software Protocol
**DO NOT USE THIS PROTOCOL IF YOU ARE LOOKING FOR SOMETHING RELIABLE, READ DISCLAMER FOR MORE INFORMATION**

This Protocol is designed to be used when you do not have any option to use something else to communicate between two AVR microcontrollers. This protocol is being developed on the Atmega328p but I recon that it should work with other AVR microcontrollers as well.

It's main features are:

  - Is simple to use.
  - Works on any pin (this version is currently limited to C pin registers, later version might work on all pin registers)
  - It doesn't change timers(Make sure both microcontrollers overflow at the same rate!)
  - Doesn't require adressing.
  - Sends and receives at the same time.
  - On startup it can wait infinitely long for connection without pausing your program.


### Version
0.0.1

### Usage (**Important!**)

This is a list with functions available to you:

    (void)    ISRMSSP()                                                     //FUNCTION THAT NEEDS TO BE PUT IN ISR()
    (void)    initMSSP(syncPin, receivePin, transmitPin, localClockSpeed)
    (void)    isMaster()
    (void)    isSlave()
    (void)    sendByte(byte)
    (uint8_t) getByte()
    (void)    stopMSSP()


Before you start main, you should add in your main file:

`#include "<avr/interrupt.h>`

`#include "MSSP.h"`

After doing that, add the following code **before** you write down your `int main()` :

    ISR(TIMER2_OVF_vect)
    {
        ISRMSSP();
    }
    
You initialize the protocol with `initMSSP(clockPin, receivePin, transmitPin, localClockSpeed);`, The pin setup is quit explanatory but you have to be careful when setting the localClockSpeed, to get the best sync with the other arduino, the best value is something that is **fully dividable by the overflow rate of timer2**. For example, when you overflow at 82000 Hz, you probably want something along  41, 82, 164, 328, etc. 

An example would be (without serial communication):
    
    #include <avr/io.h>
    #include <avr/interrupt.h>  
    #include <stdint.h>         //if not done automatically
    #include "MSSP.h"
    
    int main()
    {
        //configure pins and set the local clock rate, best value is when you can divide timer overflows per second (hz) by 2
        initMSSP(PINC2, PINC0, PINC1, 82);
        
        //If you want to be a Slave:
        isSlave(); 
        
        //If you want to be a Master
        isMaster();
        
        //infite while loop where your program runs
        while(1)
        {
            //send byte (be aware, this does not wait for byte to be send, this can be changed mid transfer!)
            sendByte(66);
            
            //gets the latest received byte (be aware, this is not a event, this will just retreive the latest received byte!)
            if(getByte() == 66){
                sendByte(163);
            }
        }
        
        return 1;
    }
    
### How does it work?

Very simple: This protocol is a semi local timed/sync protocol, it sends a full byte on its local clock/timing before sending/receiving a sync pulse to sync the both local clocks again. Why doing it like this? Well, It might be just me, but I couldn't get a clock signal over the sync pin to work, so I went with the next best thing. I fully know that this is not reliable in any way and is also very inefficient. I might try to do it with a sync signal to control the slave clock speed but as it is now, it somewhat works.

### Development
This protocol was developed and used in a project for college. We needed to create a game and one of our features was Multiplayer with 2 screens (both arduino's had a screen and a nunchuck through I2C) [Link to a video demonstrating this protocol in combination with our "game".](https://www.youtube.com/watch?v=qZhvvHVLEhM) Keep in mind that this was an older version. We couldn't use I2C to communicate between the two arduinos and SPI was also out of the question. We tried, software I2C, SPI and usart but couldn't get it to work properly with our pin setup. So I started development of this protocol. This protocol was developed with the idea: "If I can somewhat send and receive data and use this data, its fine.", so in that perspective, It wasn't developed with perfomance or even high reliability in mind.

I will continue some development on it, but if someone else can improve on, please be my guest :D.

### Todos

- Make it in a fully working Class Library (now functions are just extended, not very readable)
- Reliability stuff, it may suddenly only receive integers that can only be devided by 2 (12, 36, 32, 68, etc...)
- Changable pin registers 
- Changeable timers
- Make it faster

### Disclaimer
This library is still in alpha stadium, many things might go wrong and therefore I am not 
responsible for whatever happens while you use this application.

**DO NOT USE THIS PROTOCOL IF YOU ARE LOOKING FOR SOMETHING RELIABLE!**

Things that might happen:

- IT MAY RECEIVE BYTES CONSISTENT BUT DIFFERENT FROM WHAT YOU MEANT TO SEND (send 37, get 36, send 39, get 38, etc...)
- IT MAY RANDOMLY SEND A INCORRECT VALUE BUT IT MIGHT CHANGE BACK TO WHAT YOU WANTED TO SEND/RECEIVE
- IT MAY START SENDING INCORRECT INFORMATION AFTER A WHILE
- IT GETS DEFINATELY UNRELIABLE ON HIGH CLOCK RATES (defined in `initMSSP(x,x,x,HERE)`)!

License
----

MIT


**Free Software, Hell Yeah!**




