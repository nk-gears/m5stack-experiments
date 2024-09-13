import os, sys, io
import M5
from M5 import *
from hardware import *
import math
import time
import requests2



label0 = None
image0 = None
label1 = None
label2 = None  # New label for message on second page
circle0 = None
rotary = None
current_page = 0  # To track the current page (0 for main, 1 for message)
selected_value = -1  # To store the last rotary value

 

# Function to switch pages
def switch_page(page):
    global label0, label1, label2, circle0
    Widgets.fillScreen(0x222222)  # Clear the screen
    
    if page == 0:
        # Main Page
        label0.setText(str(rotary.get_rotary_value()))
        label0.setVisible(True)
        image0.setVisible(True)
        label1.setVisible(True)
       # circle0.setVisible(True)
        label2.setVisible(False)  # Hide the message label
    elif page == 1:
        # Message Page
        label0.setVisible(False)
        label1.setVisible(False)
   
        label2.setVisible(True)  # Show the message label
        label2.setText(f"Sel: {rotary.get_rotary_value()}")


# Button A click event
def btnA_wasClicked_event(state):
    global current_page, rotary
    if current_page == 0:
        # Switch to message page when on the main page
        current_page = 1
        switch_page(1)
    else:
        # Return to the main page when on the message page
        rotary.reset_rotary_value()
        current_page = 0
        switch_page(0)

 
def call_https_endpoint(url):
    try:
        headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        }
        
        response = requests2.get(url, headers=headers)
        
        if response.status_code == 200:
            print("HTTPS Response:", response.text)
        else:
            print("Error: HTTP Status Code", response.status_code)
    except Exception as e:
        print(f"Error making HTTPS request: {e}")
def setup():
    global label0, label1, label2, circle0, rotary

    M5.begin()
    Widgets.fillScreen(0x222222)
    Widgets.setRotation(3)

    # Main Page Widgets
    label0 = Widgets.Label("Nirmal", 96, 80, 1.0, 0xffa000, 0x222222, Widgets.FONTS.DejaVu72)
    label1 = Widgets.Label("label1", 25, 114, 1.0, 0xffffff, 0xb32a2a, Widgets.FONTS.DejaVu18)
    #circle0 = Widgets.Circle(117, 46, 15, 0xffffff, 0xffffff)

    # Message Page Widget
    label2 = Widgets.Label("Sel: 0", 25, 114, 1.0, 0xffa000, 0x222222, Widgets.FONTS.DejaVu24)
    label2.setVisible(False)  # Initially hidden

    BtnA.setCallback(type=BtnA.CB_TYPE.WAS_CLICKED, cb=btnA_wasClicked_event)

    rotary = Rotary()
    rotary.reset_rotary_value()


def loop():
    global rotary, label0, label2, current_page, selected_value
    M5.update()

    if rotary.get_rotary_status():
        # Update the rotary value on the current page
        new_value = rotary.get_rotary_value()

        if current_page == 0:
            label0.setText(str(new_value))
        elif current_page == 1:
            label2.setText(f"Sel: {new_value}")

        # Check if the value is 5 and call the HTTP endpoint
        if new_value == 5 and new_value != selected_value:
            print("Num 5 selected, calling HTTP endpoint...")
            call_https_endpoint("https://nkgears-iot-dz75mjs44a-ew.a.run.app/iot?device=Office%20UPS&state=0")

        if new_value == 6 and new_value != selected_value:
            print("Num 6 selected, calling HTTP endpoint...")
            call_https_endpoint("https://nkgears-iot-dz75mjs44a-ew.a.run.app/iot?device=Office%20UPS&state=1")            

        # Update the selected value to avoid repeated calls
        selected_value = new_value


if __name__ == '__main__':
    try:
        setup()
        while True:
            loop()
    except (Exception, KeyboardInterrupt) as e:
        try:
            from utility import print_error_msg
            print_error_msg(e)
        except ImportError:
            print("please update to latest firmware")

