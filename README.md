# Raspberry Pi Pico W + BME280 + Thingspeak
# This is the code to use if you want to use a Rasberry Pi Pico W and use a BME280 captor that can Read Temperature, Pression and Humidity. With the help of the
# Raspberry Pi Pico W and the Thingspeak services, you will be able to view and analyze data In Thingspeak. This is the first step to guide you to make your
# Own Weather Station! :) have fun!!

# IMPORTS SECTION

import machine
import utime
from machine import Pin, I2C        #importing relevant modules & classes
import bme280       #importing BME280 library
import time
import network
import urequests

#Début Network WIFI setup
ssid = 'YOUR_WIFI_SSID'
password = 'WIFI_YOUR_PASSWORD'

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(ssid, password)

# Wait for connect or fail
max_wait = 10
while max_wait > 0:
    if wlan.status() < 0 or wlan.status() >= 3:
        break
    max_wait -= 1
    print('waiting for connection...')
    time.sleep(1)

# Handle connection error
if wlan.status() != 3:
    raise RuntimeError('network connection failed')
else:
    print('connected')
    status = wlan.ifconfig()
    print( 'ip = ' + status[0] )

#Fin Network Wifi Setup
    
myHOST = 'api.thingspeak.com'
myPORT = '80'
myAPI = 'YOUR_THINGSPEAK_API_KEY'
HTTP_HEADERS = {'Content-Type': 'application/json'}

i2c=I2C(0,sda=Pin(0), scl=Pin(1), freq=400000)    #initializing the I2C method 

while True:
    bme = bme280.BME280(i2c=i2c)        #BME280 object created
    temperature = bme.values[0]         #reading the value of temperature
    pressure = bme.values[1]            #reading the value of pressure
    humidity = bme.values[2]            #reading the value of humidity
    print('Temperature: ', temperature[:-1])    #printing BME280 values
    print('Humidity: ', humidity[:-1])
    print('Pressure: ', pressure[:-3])
    print ('!About to send data to thingspeak')
    
    bme_readings = {'field1':temperature, 'field2':pressure, 'field3':humidity} 
    request = urequests.post( 'http://api.thingspeak.com/update?api_key=' + myAPI, json = bme_readings, headers = HTTP_HEADERS )  
    request.close() 
    print(bme_readings) 

    utime.sleep(600.0)
    print ('Data send to thing speak')
