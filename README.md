# Simple Universal Touchpad for Homeassistant / Lovelace
This card displays a simple universal touchpad which writes its events into helper variables. This way the events can be acted upon with any automation.

## Sample view
### Card
![card](https://github.com/user-attachments/assets/793eecfe-1c6e-4898-ab20-1a30e0743871)
### Configuration
![edit](https://github.com/user-attachments/assets/3448b2f4-05cd-4bc6-9193-026b2cd87886)

## Features
- Simple to use card
- Flexible mapping to actors trough full feature set of automations

## Generated Events
- Mouse move in x and y coordinates
- Left/right mouse click
  - When buttons are press
  - When the touchpad area is tapped
- Left Mouse button hold
  - longpress to start hold, tap to release)


# Setup

## Installation
- In HomeAssistant navigate to HACS and open the overflow menu (three dots at the top)
- Select "Custom Repositories" and enter the following into the pop-up:
  - Repository: https://github.com/DFranzen/Simple-Universal-Touchpad-for-Homeassistant/
  - Type: Dashboard
- Click Add
- In HACS click on tree dots in the row for "Simple Universal Touchpad"
- Select "Download" and confirm with "Download"
- After the download is finished, confirm "Reload" in the pop-up

## Preperation
The Touchpad card interfaces with HomeAssistant through helpers.
To Prepare for the configuration you need to create a few new helpers, which the card can use to store the events.

Open the "Helpers" Tab under "Settings" -> "Devices & Services" and Create the following helpers
- 2 input_numbers for the mouse movements (x/y), suggested names: Mouse_x, Mouse_y
- 2 input_boolean for the mouse buttons (left/right), suggested names: Mouse_ButtonLeft, Mouse_ButtonRight

## Configuration
Add the touchpad card to the dashboard and assign the newly created helpers to the fields in the configuration pop-up.

## Usage
Once the card is interacted with it writes the events into the given helpers. The input_boolen helpers are set to true, when the mouse button is pressed and the input_numbers helpers contain the last distance travelled in x/y direction.
In order to link the events written to the helpers with actors automations can be used. These could do any action in HomeAssistant. The automations should have the following properties
- The trigger is
'''
entity_id:
  - input_number.mouse_y
trigger: state
'''

- One of the actions under "Then do" should reset the helper to 0 or false:
'''
data:
  value: 0
target:
  entity_id: input_number.mouse_y
action: input_number.set_value
'''

# Example Usage
## ssh - xdotool
One of the easiest way to relay the inputs to the mouse-pointer of a computer is ssh under kde with xdotool

### Reqiurements
- The Computer should be running linux with x11-server as the window manager
- The HomeAssistant server needs ssh-access to the target computer.
  - Grant authorisation, either
    - set up password-less login from the HomeAssistant device
    - (not recommended) use sshpass infront of all ssh commands
  - Make ssh sessions permanent (Otherwise HomeAssistant needs to authenticate for every mouse movement, resulting in delays):
  Add the following to .ssh/config on the HomeAssistant device
  '''
     Host *
          ControlMaster auto
     	  ControlPath /tmp/ssh-%r@%h:%p
     	  ControlPersist yes
  '''
- 

### Preparation
Add the following shell_commands to the configuration.yaml. Substitude "user" and "targetip" with the appropriate values and modify mouse_x/mouse_y with the names you assigned to the helper variables.
'''
shell_command:
  computer_mousemove:  ssh user@targetip 'export DISPLAY=:0;xdotool mousemove_relative -- {{ ((states.input_number.mouse_x.state | float *-10 ) ) | string }} {{ ((states.input_number.mouse_y.state | float *-10 ) ) | string }}'  
  computer_mouse_leftdown:  ssh user@targetip 'export DISPLAY=:0;xdotool mousedown --clearmodifiers 1'
  computer_mouse_leftup:    ssh user@targetip 'export DISPLAY=:0;xdotool mouseup   --clearmodifiers 1'
  computer_mouse_rightdown: ssh user@targetip 'export DISPLAY=:0;xdotool mousedown --clearmodifiers 3'
  computer_mouse_rightup:   ssh user@targetip 'export DISPLAY=:0;xdotool mouseup   --clearmodifiers 3'  
'''

### Automation
Mouse move:
'''
alias: MouseMove_to_Framework
description: ""
triggers:
  - entity_id:
      - input_number.mouse_y
    trigger: state
actions:
  - action: shell_command.framework_mousemove
    data: {}
  - data:
      value: 0
    target:
      entity_id: input_number.mouse_y
    action: input_number.set_value
  - data:
      value: 0
    target:
      entity_id: input_number.mouse_x
    action: input_number.set_value
mode: single
'''

Mouse Down/up (Example for Left Down, adjust for right and up)
'''
alias: MouseLeftDown_to_Framework
description: ""
triggers:
  - entity_id:
      - input_boolean.mouse_buttonleft
    to: "on"
    trigger: state
actions:
  - action: shell_command.framework_mouse_leftdown
    data: {}
mode: single
'''

## MQTT / Node-red

### Requirements

### Setup on Node-red

### Automation

## Conditional actors
