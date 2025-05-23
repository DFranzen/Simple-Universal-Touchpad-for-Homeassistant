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
```
entity_id:
  - input_number.mouse_y
trigger: state
```

- One of the actions under "Then do" should reset the helper to 0 or false:
```
data:
  value: 0
target:
  entity_id: input_number.mouse_y
action: input_number.set_value
```

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
  ```
     Host *
          ControlMaster auto
     	  ControlPath /tmp/ssh-%r@%h:%p
     	  ControlPersist yes
  ```
- 

### Preparation
Add the following shell_commands to the configuration.yaml. Substitude "user" and "targetip" with the appropriate values and modify mouse_x/mouse_y with the names you assigned to the helper variables.
```
shell_command:
  computer_mousemove:  ssh user@targetip 'export DISPLAY=:0;xdotool mousemove_relative -- {{ ((states.input_number.mouse_x.state | float *-10 ) ) | string }} {{ ((states.input_number.mouse_y.state | float *-10 ) ) | string }}'  
  computer_mouse_leftdown:  ssh user@targetip 'export DISPLAY=:0;xdotool mousedown --clearmodifiers 1'
  computer_mouse_leftup:    ssh user@targetip 'export DISPLAY=:0;xdotool mouseup   --clearmodifiers 1'
  computer_mouse_rightdown: ssh user@targetip 'export DISPLAY=:0;xdotool mousedown --clearmodifiers 3'
  computer_mouse_rightup:   ssh user@targetip 'export DISPLAY=:0;xdotool mouseup   --clearmodifiers 3'  
```

### Automation
Mouse move:
```
alias: MouseMove_to_Laptop
description: ""
triggers:
  - entity_id:
      - input_number.mouse_y
    trigger: state
actions:
  - action: shell_command.computer_mousemove
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
```

Mouse Down/up (Example for Left Down, adjust for right and up)
```
alias: MouseLeftDown_to_Laptop
description: ""
triggers:
  - entity_id:
      - input_boolean.mouse_buttonleft
    to: "on"
    trigger: state
actions:
  - action: shell_command.computer_mouse_leftdown
    data: {}
mode: single
```

## MQTT / Node-red

### Requirements
- Node-red setup on the target computer
- Target computer reachable through node-red from the HomeAssistant

### Setup on Node-red
#### Mousemove flow
<details>

<summary>Example Node-Red configuration</summary>
Replace all the <MQTTBroker...> information with your own

```
[
    {
        "id": "3a04694b8d100522",
        "type": "mqtt in",
        "z": "9072b355de756fe9",
        "name": "",
        "topic": "input_pair/Laptop_Mouse/set",
        "qos": "2",
        "datatype": "auto-detect",
        "broker": "7715a72184178191",
        "nl": false,
        "rap": true,
        "rh": 0,
        "inputs": 0,
        "x": 280,
        "y": 1060,
        "wires": [
            [
                "524a96c3928b323b"
            ]
        ]
    },
    {
        "id": "524a96c3928b323b",
        "type": "function",
        "z": "9072b355de756fe9",
        "name": "extract coordinates",
        "func": "var factor=10\nif (Number.isInteger(msg.payload.x) && Number.isInteger(msg.payload.y))\n  factor = 1\nreturn {payload: \"\" + Math.round(-msg.payload.x*factor) + \" \" + Math.round(-msg.payload.y*factor)};",
        "outputs": 1,
        "noerr": 0,
        "initialize": "",
        "finalize": "",
        "libs": [],
        "x": 570,
        "y": 1060,
        "wires": [
            [
                "01badf34c2ebf7ec"
            ]
        ]
    },
    {
        "id": "01badf34c2ebf7ec",
        "type": "exec",
        "z": "9072b355de756fe9",
        "command": "export DISPLAY=:0;xdotool mousemove_relative -- ",
        "addpay": "payload",
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "winHide": false,
        "oldrc": false,
        "name": "",
        "x": 1050,
        "y": 1060,
        "wires": [
            [],
            [],
            []
        ]
    },
    {
        "id": "7715a72184178191",
        "type": "mqtt-broker",
        "name": "<MQTTBrokerName>",
        "broker": "<MQTTBrokerIP>",
        "port": "<MQTTPort>",
        "tls": "<MQTTBrokerTLS>",
        "clientid": "<MQTTBrokerID>",
        "autoConnect": true,
        "usetls": true,
        "protocolVersion": "4",
        "keepalive": "60",
        "cleansession": false,
        "birthTopic": "",
        "birthQos": "0",
        "birthPayload": "",
        "birthMsg": {},
        "closeTopic": "",
        "closeQos": "0",
        "closePayload": "",
        "closeMsg": {},
        "willTopic": "",
        "willQos": "0",
        "willPayload": "",
        "willMsg": {},
        "userProps": "",
        "sessionExpiry": ""
    },
    {
        "id": "<MQTTBrokerTLS>",
        "type": "tls-config",
        "name": "",
        "cert": "",
        "key": "",
        "ca": "",
        "certname": "",
        "keyname": "",
        "caname": "ca.crt",
        "servername": "",
        "verifyservercert": false,
        "alpnprotocol": ""
    }
]
```
</details>

#### Mouse Buttons

<details>

<summary>Example Node-Red configuration</summary>

Replace all the <MQTTBroker...> information with your own

```
[
    {
        "id": "68ee587de3365dea",
        "type": "mqtt in",
        "z": "9072b355de756fe9",
        "name": "",
        "topic": "Laptop",
        "qos": "2",
        "datatype": "auto-detect",
        "broker": "7715a72184178191",
        "nl": false,
        "rap": true,
        "rh": 0,
        "inputs": 0,
        "x": 270,
        "y": 260,
        "wires": [
            [
                "f63a897fbf0460ad",
                "5e5ae2d9cfad85f8",
                "f3f9502a7003f089",
                "df065cd6c88b4e8b",
                "420ff0bba7a7cbe2",
                "c9716270880d3135",
                "49a5c228e64102dc",
                "56cb45a845748ce6",
                "5323ed83dc049965",
                "40f7e260cd17ca73",
                "10f6151b0d8c6f9d",
                "ee79288130de68b1",
                "9ccff470ca2fc817",
                "2c1966a95f9e2761",
                "0b380d539e0bfc8b",
                "bb35db7c5d7dc5b3",
                "491c9a8850117c71",
                "61e87448ede9ac50",
                "f0038b2d1afc638c",
                "e428907093e90395"
            ]
        ]
    },
    {
        "id": "491c9a8850117c71",
        "type": "switch",
        "z": "9072b355de756fe9",
        "name": "MOUSE_LEFT_DOWN",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "MOUSE_LEFT_DOWN",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 590,
        "y": 840,
        "wires": [
            [
                "2415104344d7e653"
            ]
        ]
    },
    {
        "id": "61e87448ede9ac50",
        "type": "switch",
        "z": "9072b355de756fe9",
        "name": "MOUSE_LEFT_UP",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "MOUSE_LEFT_UP",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 570,
        "y": 880,
        "wires": [
            [
                "ec8d695a743ec90b"
            ]
        ]
    },
    {
        "id": "f0038b2d1afc638c",
        "type": "switch",
        "z": "9072b355de756fe9",
        "name": "MOUSE_RIGHT_DOWN",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "MOUSE_RIGHT_DOWN",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 590,
        "y": 920,
        "wires": [
            [
                "74e9b9ce8fca68e1",
                "c18dc4dfb84e51e8"
            ]
        ]
    },
    {
        "id": "e428907093e90395",
        "type": "switch",
        "z": "9072b355de756fe9",
        "name": "MOUSE_RIGHT_UP",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "eq",
                "v": "MOUSE_RIGHT_UP",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 1,
        "x": 580,
        "y": 960,
        "wires": [
            [
                "7585f81886584946",
                "c18dc4dfb84e51e8"
            ]
        ]
    },
    {
        "id": "7585f81886584946",
        "type": "exec",
        "z": "9072b355de756fe9",
        "command": "export DISPLAY=:0;xdotool mouseup --clearmodifiers 3",
        "addpay": "",
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "winHide": false,
        "oldrc": false,
        "name": "",
        "x": 1060,
        "y": 960,
        "wires": [
            [],
            [],
            []
        ]
    },
    {
        "id": "74e9b9ce8fca68e1",
        "type": "exec",
        "z": "9072b355de756fe9",
        "command": "export DISPLAY=:0;xdotool mousedown 3",
        "addpay": "",
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "winHide": false,
        "oldrc": false,
        "name": "",
        "x": 1020,
        "y": 920,
        "wires": [
            [],
            [],
            []
        ]
    },
    {
        "id": "ec8d695a743ec90b",
        "type": "exec",
        "z": "9072b355de756fe9",
        "command": "export DISPLAY=:0;xdotool mouseup --clearmodifiers 1",
        "addpay": "",
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "winHide": false,
        "oldrc": false,
        "name": "",
        "x": 1060,
        "y": 880,
        "wires": [
            [],
            [],
            []
        ]
    },
    {
        "id": "2415104344d7e653",
        "type": "exec",
        "z": "9072b355de756fe9",
        "command": "export DISPLAY=:0;xdotool mousedown --clearmodifiers 1",
        "addpay": "",
        "append": "",
        "useSpawn": "false",
        "timer": "",
        "winHide": false,
        "oldrc": false,
        "name": "",
        "x": 1070,
        "y": 840,
        "wires": [
            [],
            [],
            []
        ]
    },
    {
        "id": "7715a72184178191",
        "type": "mqtt-broker",
        "name": "<MQTTBrokerName>",
        "broker": "<MQTTBrokerIP>",
        "port": "<MQTTPort>",
        "tls": "<MQTTBrokerTLS>",
        "clientid": "<MQTTBrokerID>",
        "autoConnect": true,
        "usetls": true,
        "protocolVersion": "4",
        "keepalive": "60",
        "cleansession": false,
        "birthTopic": "",
        "birthQos": "0",
        "birthPayload": "",
        "birthMsg": {},
        "closeTopic": "",
        "closeQos": "0",
        "closePayload": "",
        "closeMsg": {},
        "willTopic": "",
        "willQos": "0",
        "willPayload": "",
        "willMsg": {},
        "userProps": "",
        "sessionExpiry": ""
    },
    {
        "id": "<MQTTBrokerTLS>",
        "type": "tls-config",
        "name": "",
        "cert": "",
        "key": "",
        "ca": "",
        "certname": "",
        "keyname": "",
        "caname": "ca.crt",
        "servername": "",
        "verifyservercert": false,
        "alpnprotocol": ""
    }
]
```
</details>

          
### Automation

```
alias: Mouse Move to MQTT
description: ""
triggers:
  - entity_id:
      - input_number.mouse_y
    trigger: state
actions:
  - data:
      qos: 0
      retain: false
      topic: input_pair/Laptop_Mouse/set
      payload: >
        {{ "{\"x\": " + (states.input_number.mouse_x.state) | string + ", \"y\":
        " + (states.input_number.mouse_y.state) | string + "}" }}
    action: mqtt.publish
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
```

#### Mouse Button
```
alias: Mouse Left button Down to MQTT
description: ""
triggers:
  - entity_id:
      - input_boolean.mouse_buttonleft
    to: "on"
    trigger: state
conditions:
  - condition: state
    entity_id: input_select.video_gerat
    state: Laptop
actions:
  - data:
      topic: Laptop
      payload: MOUSE_LEFT_DOWN
    action: mqtt.publish
mode: single
```

## Conditional actors
The flexibility of the automations allows for conditional actors, meaning the events are send to different devices (even via different protocols) depending on a condition.

### Example
Here is an example automation of using an input_select condition to send the buttonleft event to either a laptop or a gaming computer.

```
description: ""
mode: single
triggers:
  - entity_id:
      - input_boolean.mouse_buttonleft
    to: "on"
    trigger: state
actions:
  - choose:
      - conditions:
          - condition: state
            entity_id: input_select.active_playback_device
            state: Laptop
        sequence:
          - action: shell_command.laptop_mouse_leftdown
            metadata: {}
            data: {}
      - conditions:
          - condition: state
            entity_id: input_select.active_playback_device
            state: Gaming
        sequence:
          - action: shell_command.gaming_mouse_leftdown
            metadata: {}
            data: {}

```
