## ECE 4180 Final Project- mbed Crypt
##### by Andrew McEntaggart, Keshav Shenoy, Sultan Sayedzada

For the final project, we designed and built a rhythm-based movement evasion game, inspired by the indie dungeoner [Crypt of the Necrodancer](https://store.steampowered.com/app/247080/Crypt_of_the_NecroDancer/). We incorporated some key features of Crypt of the Necrodancer into our mbed version. For instance, music plays throughout the game. The player character can move once per beat, which follows the music and is also indicated by a flashing LED on the mbed. Enemies, scattered randomly throughout the floor, can kill the player and end the game if moved into. After reaching the staircase on the first floor and evading the boss on the second, the game ends.

Mbed Crypt displays on a [uLCD board](https://os.mbed.com/users/4180_1/notebook/ulcd-144-g2-128-by-128-color-lcd/) and takes player input from a Bluetooth module connected to a smartphone. It also includes a two-player mode, where a second player traverses the map alongside the first and uses the other four buttons in the Bluefruit input on the smartphone to move. Screen captures of the two levels are shown below (the player character is the red music note). A demo can be watched [here](https://youtu.be/oX4B0y-FU48).

![uLCD screencap of floor 1](/proj1.jpg) ![uLCD screencap of floor 2](/proj2.jpg)

### Wiring

The mbed interfaces with a [uLCD display](https://os.mbed.com/users/4180_1/notebook/ulcd-144-g2-128-by-128-color-lcd/), an [Adafruit Bluetooth UART Friend](https://os.mbed.com/users/4180_1/notebook/adafruit-bluefruit-le-uart-friend---bluetooth-low-/), and a Class D Audio Amp connected to a speaker.

mbed | Audio Amp | Speaker
--- | --- | ---
VOUT | PWR+ | 
GND | PWR-, OUT- | INPUT-
P25 | IN- | 
| | OUT+ | INPUT+

The audio amp takes power from the Vout pin on the mbed, which draws less current and makes the PWM sound less obnoxious.

mbed | UART
--- | ---
GND | GND, CTS
VU | VIN
p28 | RXI
p27 | TXO

mbed | uLCD | uLCD Header Socket
--- | --- | ---
VU | +5V | +5V
p10 | TX | RX
p9 | RX | TX
GND | GND | GND
p11 | RES | RES

### Code

The game uses multi-threading to control the speakers, poll the Bluetooth serial interface, and run the timer simultaneously. The `clk` method, for instance, is attached to the timer thread and updates a variable `counter` which is later used to determine if an action input from the Bluetooth is valid or not.
```
void clk() {
    // Game code waits for input on a timer
    beat = 1;
    // Have clock increment a counter from 0 upZ to 100 (wait time depends on song [level])
    for(;;) {
        Thread::wait(levelTimes());
        counter++;
        if (counter > 100) {
            counter = 1;
        }
        //reset ability to make action on half beat
        if (counter == 50) {
            processEnemyAction();
            actLock = 1;
            actLock2 = 1;
        }
        //indicate when a press is on beat. May need to alter according to calibration
        if (counter == 25 || counter == 75) {
            beat = !beat;
        }
    }
}

...

void actionable(int i) {
    if (actLock) {
        actM.lock();
        actLock = 0;
        actM.unlock();
        if (counter <= 25 || counter >= 75) {
            currAction = i;
        } else {
            currAction = 0;
        }
        processAction();
    }
}
```

The action is only processed by the code if it was made sufficiently close to the beat hit, i.e. if it was received within a quarter of a period of the beat change.

The full repository is published [here](https://os.mbed.com/users/amcentag/code/crypt_mbed/).
