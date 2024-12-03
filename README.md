#  Sommaire
*  [LCD](#LCD)
*  [SDA_SCL_Multiple](#SDA_SCL_Multiple)


# LCD
L’écran LCD 0802 du kit KS0522



```bash
sudo pip3 install adafruit-circuitpython-charlcd --break-system-packages
```

![pin correspondance](./images/lcd_gpio.png)



[voir la documentation](https://docs.circuitpython.org/projects/charlcd/en/latest/index.html)

```python
import board
import digitalio
import adafruit_character_lcd.character_lcd as CharLCD
from time import sleep

## board.D1 = GPIO 1, board.D2 = GPIO 2, etc.
## Mettez ici les valeurs qui correspondent à vos connexions
RS = digitalio.DigitalInOut(board.D1)
E = digitalio.DigitalInOut(board.D1)
DB7 = digitalio.DigitalInOut(board.D1)
DB6 = digitalio.DigitalInOut(board.D1)
DB5 = digitalio.DigitalInOut(board.D1)
DB4 = digitalio.DigitalInOut(board.D1)

cols = 8
rows = 2

lcd = CharLCD.Character_LCD(RS,E,DB4,DB5,DB6,DB7,cols,rows)

lcd.message = "hello"
sleep(1)
lcd.cursor_position(5,1)
lcd.message = "bye"
sleep(1)
lcd.clear()

```

Une manière simple de contrôler le contraste de l’écran est de brancher la broche 3 sur le signal d’un potentiomètre.

# SDA_SCL_Multiple

Vous pouvez connecter plusieurs composants utilisant SDA et SCL en même temps. Il suffit d'utiliser leur adresse respective lors de la communication.

Faites `i2cdetect -y 1` pour connaître leurs adresses et vérifier s'ils sont bien connectés.

Ici, le code utilise la led 8x8 en même temps que le potentiomètre:

```python
import board
import busio
import pigpio
from time import sleep
from adafruit_ads1x15.ads1115 import P0
from adafruit_ads1x15.ads1115 import ADS1115
from adafruit_ads1x15.analog_in import AnalogIn
import adafruit_bus_device.i2c_device as i2c_device


def allumeLed88(module,total):
    number = int(total)
    nbRows = (number // 8) + 1

    # Allumer les rangées pleines
    for row in range(0, (nbRows-1)* 2,2):
        module.write(bytes([row,255])) 

    # Allumer la rangée partiellement pleine
    module.write(bytes([(nbRows -1 ) * 2,transform(total)])) 

    # Éteindre les autres
    for row in range((nbRows)* 2, 16 ,2):
        module.write(bytes([row,0])) 
    
def transform(numb):
    nbAllume = int(numb) % 8
    if(numb > 63.5):
        return 0b11111111
    match nbAllume:
        
        case 1:
            return 0b00000001
        case 2:
            return 0b00000011
        case 3:
            return 0b00000111
        case 4:
            return 0b00001111
        case 5:
            return 0b00011111
        case 6:
            return 0b00111111
        case 7:
            return 0b01111111

    return 0b00000000
       

i2cBus = busio.I2C(board.SCL, board.SDA)

led88 = i2c_device.I2CDevice(i2cBus,0x70)

# Init ADS1115
# On utilise le même bus, c'est seulement l'adresse qui change
ads = ADS1115(i2cBus,2/3)
data = AnalogIn(ads, P0)

# Init GPIO
pi = pigpio.pi()

MAX = 4.6
MIN = 0.2
# Test
while True:
    try:
        curVolt = data.voltage
        print(curVolt)
        sleep(0.05)
        allumeLed88(led88,min(max((curVolt-MIN)/MAX * 64,0),63.99))

        

    except KeyboardInterrupt:
        for row in range(0,16,2):
            led88.write(bytes([row,0])) # Écrire les données à l'adresse
        exit()

```