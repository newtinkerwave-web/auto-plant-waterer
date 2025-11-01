# auto-plant-waterer
Code for Automatic Plant Waterer using MOSFET and a moisture sensor.
# ===============================================================
# Raspberry Pi Automatic Plant Waterer
# Author: TinkerWaves
# Description:
# Monitors soil moisture using an analog sensor via MCP3008,
# and activates a water pump through a MOSFET when the soil is dry.
# ===============================================================

import time
import spidev  # SPI communication with MCP3008
import RPi.GPIO as GPIO  # Control Raspberry Pi pins

# -----------------------------
# GPIO Pin Configuration
# -----------------------------
PUMP_PIN = 18  # GPIO pin controlling MOSFET gate (BCM numbering)
CHANNEL = 0    # MCP3008 channel connected to the moisture sensor

# -----------------------------
# Moisture Threshold
# -----------------------------
# The higher the value, the dryer the soil.
# You might need to calibrate this based on your sensor and soil.
DRY_THRESHOLD = 600  # adjust after testing (range: 0-1023)

# -----------------------------
# Setup SPI for MCP3008
# -----------------------------
spi = spidev.SpiDev()
spi.open(0, 0)  # Open SPI bus 0, device 0
spi.max_speed_hz = 1350000

# -----------------------------
# GPIO Setup
# -----------------------------
GPIO.setmode(GPIO.BCM)
GPIO.setup(PUMP_PIN, GPIO.OUT)
GPIO.output(PUMP_PIN, GPIO.LOW)  # Make sure pump starts OFF


# -----------------------------
# Function: Read MCP3008 Channel
# -----------------------------
def read_channel(channel):
    """
    Reads an analog value from the specified MCP3008 channel (0-7).
    Returns integer between 0 and 1023.
    """
    adc = spi.xfer2([1, (8 + channel) << 4, 0])
    data = ((adc[1] & 3) << 8) + adc[2]
    return data


# -----------------------------
# Function: Water Plant
# -----------------------------
def water_plant(duration=3):
    """
    Turns on pump for specified seconds.
    """
    print(f"Watering plant for {duration} seconds...")
    GPIO.output(PUMP_PIN, GPIO.HIGH)
    time.sleep(duration)
    GPIO.output(PUMP_PIN, GPIO.LOW)
    print("Watering complete.")


# -----------------------------
# Main Loop
# -----------------------------
try:
    print("Automatic Plant Waterer Running...")
    while True:
        moisture_value = read_channel(CHANNEL)
        print(f"Moisture reading: {moisture_value}")

        if moisture_value > DRY_THRESHOLD:
            print("Soil is dry! Activating pump...")
            water_plant()
        else:
            print("Soil is moist. No watering needed.")

        time.sleep(2)  # wait 2 seconds before checking again

except KeyboardInterrupt:
    print("\n Program stopped manually.")
finally:
    GPIO.cleanup()
    spi.close()
    print("GPIO and SPI cleaned up.")
