![Logo](admin/text2command.png)
ioBroker text2command adapter
=================

[![NPM version](http://img.shields.io/npm/v/iobroker.text2command.svg)](https://www.npmjs.com/package/iobroker.text2command)
[![Downloads](https://img.shields.io/npm/dm/iobroker.text2command.svg)](https://www.npmjs.com/package/iobroker.text2command)
[![Tests](https://travis-ci.org/ioBroker/ioBroker.text2command.svg?branch=master)](https://travis-ci.org/ioBroker/ioBroker.text2command)

[![NPM](https://nodei.co/npm/iobroker.text2command.png?downloads=true)](https://nodei.co/npm/iobroker.text2command/)

This adapter can convert normal sentences, like *'Switch light in kitchen on'* to specific command and sets the state *'adapter.0.device.kitchenLight'* to **true**.

This adapter make no sense to be activated standalone. It should be used with other adapters like telegram or Android app **iobroker.vis**.

To execute command, write state **text2command.<INSTANCE>.text** with sentence. You will always get the answer in **text2command.<INSTANCE>.response**.

If you define **Answer to ID**, the answer will be written in this ID too. This required for e.g. to realise the voice acknowledges. 

You can send a message via messagebox from javascript. The answer will come in the message back:

```
sendTo('text2command', 'Switch light in kitchen on', function (err, response) {
    console.log('Response is: ' + response);
});
```

Regular expressions can be used, like: ```/^light\son|^lamp\son/```. Regular expressions are always case insensitive.

To use "Switch on/off by function" you should care of functions. 

Keywords work as following:
- keywords are divided by space
- all keywords must present in a sentence to trigger a rule: e.g. keyword: ```light on``` will trigger on ```switch light on```, ```make light on everywhere``` and do not trigger on ```switch on```, ```make light```.
- one keyword can has many forms. Variations of keyword must be divided by "/". E.g. keywords: ```switch/make/do light on/true``` will trigger on: ```do light true```, ```make please light on```.
- if keyword can come in many cases(nom, gen, accusative, plural, ...) they all must be listed as variations, like: ```switch light/lights on```. 

Following functions will be interpreted as 

enum.functions:

**enum.functions.light** (Licht | Свет):
- roles - level.dimmer
- roles - switch.light

**enum.functions.backlight** (Beleuchtung | Подсветка):
- roles - level.backlight
- roles - switch.backlight

**enum.functions.blinds/shutter** (Rolladen | Жалюзи/окна)
- roles - level.blind
- roles - switch.blind

**enum.functions.curtain** (Vorhänge | Шторы)
- roles - level.curtain
- roles - switch.curtain

**enum.functions.heating** (Heizung | Отопление/Подогрев)
- roles - level.temperature
- roles - switch.temperature

**enum.functions.music** (Musik | Музыка)
- roles - button.play
- roles - button.stop / button.pause

**enum.functions.alarm/security** (Alarmanlage / Alarm | Охрана)
- roles - switch.security

**enum.functions.lock** (Schloß / Schloss | Замок)
- roles - switch.open
- roles - switch.lock

Following rooms are supported:

| key word in phrase    | Possible enum.rooms in english  | in german                | in russian             |
|-----------------------|---------------------------------|--------------------------|------------------------|
| everywhere            | everywhere                      | -                        | -                      |
| living                | livingroom                      | wohnzimmer               | зал                    |
| bedroom               | bedroom/sleepingroom            | schlafzimmer             | спальня                | 
| bath                  | bathroom/bath                   | badezimmer/bad           | ванная                 |
| working/office        | office                          | arbeitszimmer            | кабинет                |
| kids/child/nursery    | nursery                         | kinderzimmer             | детская                |
| guets wc/guest closet | guestwc                         | gästewc                  | гостевой туалет        |
| wc/closet             | wc                              | wc                       | туалет                 |   
| floor/enter           | floor                           | diele/gang/flur          | коридор/прихожая       |
| kitchen               | kitchen                         | küche/kueche             | кухня                  |
| balcony/terrace/patio | terrace                         | balkon/terrasse          | терасса/балкон         |
| dinning               | dinningroom                     | esszimmer                | столовая               |
| garage                | garage                          | garage                   | гараж                  |
| stair                 | stairs                          | trepe/treppenhaus        | лестница               |
| garden                | garden                          | garten                   | сад                    |
| court/yard            | court                           | hof                      | двор                   |
| guest room            | guestroom                       | gästezimmer              | гостевая               |
| attic                 | attic                           | speicher                 | кладовка               |
| roof                  | roof                            | dachstuhl                | крыша                  |
| terminal              | terminal                        | anschlussraum            | сени                   |
| wash room             | washroom                        | waschraum                | прачечная              |
| heat room             | heatroom                        | heatingroom/heizungsraum | котельная              |
| hovel                 | hovel                           | schuppen/scheune         | сарай                  |
| summer house          | summerhouse                     | gartenhaus               | теплица                |

You can use patterns in acknowledges: 
- %s : value
- %u : unit
- %n : name (planned!)

Following commands are supported:

#### What time is it?
Answer: 14:56 (current time)

#### What is your name?
Answer is customizable. Default: ```My name is Alpha```

#### What is the outside temperature?
User must specify the state ID, where to read outside temperature.
Answer is customizable. Default: ```Outside temperature is %s %u```
**%s** will be replaced by temperature, rounded to integer. **%u** will be replaced by units of this state or by system temperature units. 

#### What is the inside temperature?
User must specify the state ID, where to read inside temperature.
Answer is customizable. Default: ```Inside temperature is %s %u```
**%s** will be replaced by temperature, rounded to integer. **%u** will be replaced by units of this state or by system temperature units. 

#### Switch on/off by function
This command reads information from enums. It uses **enum.functions** to find type of device (e.g light, alarm, music) and **enum.rooms** to detect room name.

Example in german:
![Enums](img/enums.png)

Key words to switch on are: *switch on*, e.g. ```switch rear light in bath on``` 

Key words to switch off are: *switch off*, e.g. ```switch light in living room off``` 

Answer will be generated automatically if desired: ```Switch off %function% in %room%```, where %function% and %room% will be replaced by found device type and location. 

Command accept the numeric value too. It has priority, e.g. in command ```switch light off in living room on 15%``` the light will be set to 15% and not in *off* state.

You can define default room in []. E.g ```switch the light on[sleepingroom]```

#### Open/close blinds
This command reads information from enums. It uses **enum.functions.blind** to find type blinds or shutter and **enum.rooms** to detect room name.

Key words to move blinds up are: *blinds up*, e.g. ```set blinds up in sleeping room``` 

Key words to move blinds down are: *blinds down*, e.g. ```move blinds down in office``` 

You can specify the exactly position of blind in percent, e.g. ```move blinds to 40 percent in office```

Answer will be generated automatically if desired: ``` in %room%```, where %room% will be replaced by found device type and location. 

#### Switch something on/off
User must specify state ID of device, which must be controlled and value, which must be written. 

You should create rule for every position (e.g. for *on* and for *off*).
  
Answer is customizable. Default: ```Switched on```

E.g.: 

- ```Deactivate alarm```, Object ID: ```hm-rpc.0.alarm```, Value: ```false```, Answer: ```Alarm is deactivated/Deactivated```. In this case the answer will be randomized between *Alarm is deactivated* and *Deactivated*.
- ```Activate alarm```, Object ID: ```hm-rpc.0.alarm```, Value: ```true```, Answer: ```Alarm is activated/Activated/Done``` . In this case the answer will be randomized between *Alarm is activated*, *Activated* and *Done*.

*Deactivate* must be first in the list, because it is longer.

You can use float values in the control commands. If some numeric value will be in the text it will be used as control value and the predefined value will be ignored.

E.G. for rule for rule:

- ```Set light level```, Object ID: ```hm-rpc.0.light.STATE```, Value: ```10```, Answer: ```Level set to %s%```.

If command is like ```Set light level to 50%```, so into the ```hm-rpc.0.light.STATE``` will be written 50 and answer will be ```Level set to 50%```.

If command is like ```Set light level```, so into the ```hm-rpc.0.light.STATE``` will be written 10 and answer will be ```Level set to 10%```.

#### Ask about something
User must specify state ID of device, which value will be read. 
This template will answer with information from some state.

E.g.:

- ```windows opened```, Object ID: ```javascript.0.countOpenedWindows```, Acknowledge: ```Actual %s windows opened```
- ```temperature sleeping room```, Object ID: ```hm-rpc.0.sleepingRoomSensor.TEMPERATURE```, Acknowledge: ```Actual temperature in sleeping room is %s %u/%s %u```. In this case the answer will be randomized between *Actual temperature in sleeping room is %s %u* and *%s %u*.

#### Send text to state
You can write some text into state. User must specify state ID to write text into it. 

E.g. rule: ```email [to] wife```, Object ID: ```javascript.0.emailToWife```, Acknowledge: ```Email sent```
Text: *Send email to my wife: I will be late*. Adapter looks for the last word from key words (in this case *wife*), 
extracts text from the next word (in this case *I will be late*) and writes this text into *javascript.0.emailToWife*. 
Word *to* is not required to trigger the rule, but will be removed from text.   

### You are good (Just for fun)
Answer is customizable. Default: ```Thank you``` or ```You are welcome```

### Thank you (Just for fun)
Answer is customizable. Default: ```No problem``` or ```You are welcome```

# ToDo
- in Russian male and female answers.

## Changelog
### 1.0.2 (2016-07-22)
* (bluefox) fix error with detection of numeric values

### 1.0.1 (2016-06-01)
* (bluefox) fix: send text command

### 1.0.0 (2016-05-05)
* (bluefox) replace special chars in input text: #'"$&/\!?.,;:(){}^

### 0.1.10 (2016-03-20)
* (bluefox) fix double pronunciation of some answers

### 0.1.9 (2016-03-20)
* (bluefox) ignore spaces

### 0.1.8 (2016-03-15)
* (bluefox) fix error with enums

### 0.1.7 (2016-03-12)
* (bluefox) implement "say something"

### 0.1.6 (2016-02-24)
* (bluefox) fix temperature

### 0.1.5 (2016-02-23)
* (bluefox) fix russian outputs

### 0.1.4 (2016-02-22)
* (bluefox) fix russian outputs

### 0.1.3 (2016-02-21)
* (bluefox) round temperature in answers

### 0.1.2 (2016-02-21)
* (bluefox) implement russian time

### 0.1.1 (2016-02-19)
* (bluefox) check invalid commands

### 0.1.0 (2016-02-19)
* (bluefox) fix problem with controlling of channels
* (bluefox) enable write JSON as argument 

### 0.0.3 (2016-02-14)
* (bluefox) remove unused files

### 0.0.2 (2016-02-10)
* (bluefox) extend readme

### 0.0.1 (2016-02-09)
* (bluefox) initial commit

## Install

```iobroker add text2command```

## License

The MIT License (MIT)

Copyright (c) 2014-2016, bluefox<dogafox@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
