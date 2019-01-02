# Virtual-Joystick-using-Libusb-and-Uinput
Demonstrating how to use a kernel space virtual joystick into user space. This also demonstrates how to convert raw binary USB data to something readable for the program. 

Implementation of uinput:
Implementing uinput is essential to this program because it is necessary to be able to manipulate inputs in the user space. By implementing uinput, we can create commands in the program to press a specific key or button (in this case the middle button on chompapp) to move a joystick/mouse (in this case, moving left and right joysticks). 
The way user input is implemented is by first implementing a file descriptor, in our case called uinp_fd, and a structure that implements a uinput device called uinp. The file descriptor opens the /dev/uinput output file, where uinput emulates a user input device. The input output control function implements the buttons and the two axes to be recognized by the uinput device. Then set up the uinp device with the appropriate product and vendor id along with the BUS_USB type and then write the device. By implementing the uinput, the device is able to push specific key values onto user space. E.g. press the keyboard key, move a joystick, etc.

Implementation of LibUSB:
Implementing the LibUSB functions will allow the program to retrieve the inputs, as well as the current state of the device. Since this program uses an interrupt transfer, we should expect the device to transfer data when it is applied. To get the data a device handle is initialized and opened with the product ID and the vendor ID and claim the interface to retrieve transfers. Since we want the device to run all the time a while(1) loop was created and under the loop the function libusb_interrpt_transfer outputs data on a data buffer. We can then access current information with the data buffer.

Converting raw data to user space input:
After implementing both the uinput device and the libusb device we want the usb device raw data to be translated to a user space input. From what we gathered from the device description we know the first two least significant bits represent the y-axis, the next two least significant bits represent the x-axis and the next least significant bit represents the button state. By converting the raw data into binary we can extract the bits to their representative state value. 
From there we can implement an emit function that injects and input event to actually submit and record events. The functions just input the type, code, and value of each event and writes the event to the uinput file descriptor. 
Using the xState, we want to change the absolute location (EV_ABS) of the x axis (ABS_X) to -32767 (val) if x = 1, 32767 if x = 3 and 0 if x=2. For the y axis (ABS_Y) we want to change the absolute location (EV_ABS) to 32767 if y=1, -32767 if y =3, and 0 if y = 2. If the Button state is 0, the button (BTN_A) value should be 0, otherwise if button state is 1 button value should be 1.
This summarizes the translation process of converting raw data to user input.

Testing: 
The way the code was tested was by creating an error function to see if specific libusb errors occurred and printing errors if specific uinput setup errors occurred. This will help located errors at a specified point. To test the overall code: sudo ./chompapp, sudo usbip -a 127.0.0.1 1-1, make, and sudo ./chompdrv, jstest /dev/input/js0 were executed in that order to and it was checked to see if the joystick was working.
