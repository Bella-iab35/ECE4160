---
# Lab 1A #
---
## Prelab 1A##

I already had the Arduino IDE installed, so as suggested by the lab instructions, that is the IDE I used to do this lab and future lab. The only thing I had to do for this part's prelab was to install the [json file](https://learn.sparkfun.com/tutorials/artemis-development-with-the-arduino-ide/setting-up-the-arduino-ide) for the Sparkfun Apollo boards manager and then add them to the Arduino IDE settings where it asks for "Additional boards manager URLS." With that linked, I was able to install the needed board manager so that I could connect to the SparkFun RedBoard Artemis Nano.
## Tasks ##

#### Blink ####

#### Serial ####

#### Analog Reading - Temperature Sensing ####

#### PDM - Microphone Output ####

- Blink
	- Explain how code works?
	- Note if the baud rate was slowed down
	- Video from phone of the board blinking
- Serial
	- Explain code
	- Check baud rate: 115200
	- Screen record (if there's anything physical happening, record and split screen)
- analog read
	- Code
	- Record measurements and also me holding the chip -> split screen
	- serial monitor shows raw temps
- Microphone Output
	- purpose and code
	- splitscreen with output and me speaking to change frequency. If output is not video, I can personally animate it

---
# Lab 1B #
---
## Prelab 1B ##

#### In the Command Window... ####
This prelab focused on setting up the python environment that the Sparkfun Artemis was going to establish a communication channel with. I already had a compatible version of Python 3 installed (3.12.3) that fell into the range of versions (Python 3.10 - 3.13) that avoided async issues with Bleak and the lab codebase.

![[Screenshot 2026-01-20 at 1.59.48 PM.png]]

With python already installed, I then installed venv and, within my project directory for ECE 4160, created "FastRobots_ble," the virtual environment where I will end up working on the python code within Jupyter Lab. 

```
python3 -m pip install --user virtualenv

cd [Project Directory]

python3 -m venv FastRobots_ble
```

If the virtual environment was deactivated (via the deactivate command), I would run:

```
source FastRobots_ble/bin/activate
```

Inside the environment, I had to first install all needed python packages, which was done via this command:

```
pip install numpy pyyaml colorama nest_asyncio bleak jupyterlab
```

One last thing needed to be downloaded into my project directory: the [codebase](https://fastrobotscornell.github.io/FastRobots-2026/labs/ble_robot_1.4.zip) that includes both the python and arduino code for this portion of the lab. With everything that is required downloaded and installed into my directory or virtual environment, I started my Jupyter server (note that I had to use Jupyter lab with a capital J due to being on macOS).

```
Jupyter lab
```

#### In the Arduino IDE... ####

It was time to connect the Artemis board to my computer, so first I had to install ArduinoBLE from the library manager and then compile and upload the sketch ble_arduino.ino provided from the ble_arduino directory from the previously downloaded codebase. This will lead to the board's MAC address being printed to the serial monitor.
## Configurations ##

![[MACaddress.png]]
![[uuid4 1.png]]
![[uuidsmatch.png]]
![[connected.png]]

## Tasks ##

#### Echo ####

```
#Send string
ble.send_command(CMD.ECHO,"Hello World")
```


```
// Append to tx_estring_value
tx_estring_value.clear();
tx_estring_value.append("Robot says -> ");
tx_estring_value.append(char_arr);
tx_estring_value.append(" :)");
tx_characteristic_string.writeValue(tx_estring_value.c_str());

// Print to Serial Monitor
Serial.println(char_arr);
```



#### Send Three Integers ####

```
ble.send_command(CMD.SEND_THREE_FLOATS, "1.0|-2.0|5.7")
```


#### GET_TIME_MILLIS ####

```
ble.send_command(CMD.GET_TIME_MILLIS, "")
```

```
tx_estring_value.clear();
tx_estring_value.append("T:");
tx_estring_value.append((int) millis());
tx_characteristic_string.writeValue(tx_estring_value.c_str());
```

#### Notification Handler ####

```
def notification_handler(uuid, char_str):
    s = ble.bytearray_to_string(char_str)
    time = s[2:]
    print("Current time is: " + time + " ms")
    
ble.start_notify(ble.uuid['RX_STRING'], notification_handler)
```
#### Loop Getting Current Time ####

```
ble.send_command(CMD.LOOP, "")
```

```
// Variables
unsigned long startTime;
startTime = millis();
int count;
count = 0;

// Loop
while ((millis()-startTime) <= 5000){
	tx_estring_value.clear();
	tx_estring_value.append("T:");
	tx_estring_value.append((int) millis());
	tx_characteristic_string.writeValue(tx_estring_value.c_str());
	count++;
}

// Print number of timestamps that was looped through
Serial.print("Number of timestamps: ");
Serial.println(count);
```
#### Storing and Sending Time Data ####

```
timeStamps = []

def get_times(uuid, char_str):
    s = ble.bytearray_to_string(char_str)
    time = int(s[2:])
    timeStamps.append(time)
```

```
while ((millis()-startTime) <= 5000 && count < MAX_MSG_SIZE){
	...
	timeStamps[count] = millis();
	...
}
```

```
tx_estring_value.clear();
for (int time:timeStamps){
	tx_estring_value.append(time);
	tx_characteristic_string.writeValue(tx_estring_value.c_str());
}
```
#### Storing and Getting Temperature Readings and Time ####

```
timeStamps = []

temps = []

def get_temp(uuid, char_str):
    s = ble.bytearray_to_string(char_str)
    time, temp = s.split(":")
    timeStamps.append(time)
    temps.append(temp)
    
```

```
while ((millis()-startTime) <= 5000 && count < MAX_MSG_SIZE){
	time = millis();
	temp = getTempDegF();
	tx_estring_value.clear();
	tx_estring_value.append((int) time);
	tx_estring_value.append(":");
	tx_estring_value.append((int) temp);
	tx_characteristic_string.writeValue(tx_estring_value.c_str());
	temperatures[count] = temp;
	timeStamps[count] = time;
	count++;
}
```

```
tx_estring_value.clear();

for (int i = 0; i < MAX_MSG_SIZE; i++){
	tx_estring_value.append(timeStamps(i));
	tx_estring_value.append(":")
	tx_estring_value.append(temperatures(i));
	tx_characteristic_string.writeValue(tx_estring_value.c_str());
}
```
#### Difference in Data Recording Methods... ####



- When going through the demo, laptop asked if I wanted to give the terminal access to bluetooth


