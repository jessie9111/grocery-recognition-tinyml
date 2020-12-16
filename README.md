# grocery-recognition-tinyml
Final Project for CS249r: Tiny Machine Learning


### Testing out the ArduCam interfacing with Arduino nano 33 BLE

**Camera Interfacing With Arduino**

For interfacing the Arducam and the microcontroller, you will use the breadboard and use the connectors to interface them. Please refer the tinyML book chapter 9 for the pin connection from the ArduCam to the Arduino Nano 33 BLE or as shown below
![pin out diagram](media/arducam.png)

Now, you need to install the camera drivers for ArduCam. The camera driver is a firmware (software) that will allow exposing certain functionality of the hardware (camera) to a higher level software stack (in this case, its the TensorFlow Lite Micro). For example, to detect a person, the camera's image has to be read from the sensor buffer (residing in the ArduCam) and copied to the memory in the microcontroller. The camera's output is a JPEG, and to pass the image to the neural network model (remember the input to the neural network model is an image), there are extra processing steps. First, the JPEG image needs to be decoded to raw pixel values, which the neural network model expects.  Once the raw pixel values are decoded, additional preprocessing steps are performed such as converting to cropping and converting it to grayscale to output values to the serial monitor.

Luckily, we don't have to write the camera driver or the JPEG decoder ourselves, and it is provided by the camera vendor/other developers. To download the camera driver and the JPEGDecode library, follow the steps below:

1. Download the [Arducam library](https://github.com/ArduCAM/Arduino) to interface the camera with the Arduino. Extract the Zip file, add the camera driver and choose the right configuration for Arducam 5MP Plus:<br>

    ```Sketch -> Include Library -> Add .Zip Library -> <Path to the extracted Zip file> ``` <br>

    ``` open Arduino/libraries/ArduCAM/memorysaver.h ``` <br>
    ```
    //#define OV2640_MINI_2MP
    //#define OV3640_MINI_3MP
    //#define OV5642_MINI_5MP
    //#define OV5642_MINI_5MP_BIT_ROTATION_FIXED
    //#define OV2640_MINI_2MP_PLUS
    #define OV5642_MINI_5MP_PLUS
    //#define OV5640_MINI_5MP_PLUS 
    ```


2. Install the JPEGDecoder library from the Arduino IDE. <br>
    ``` Tools -> Manage Libraries -> search for "JPEGDecoder" ``` <br>
    After installing, go to ``` open /Arduino/libraries/JPEGDecoder/src/User_Config.h ``` and make sure to comment
   
   ```#define LOAD_SD_LIBRARY``` 
   
   ```#define LOAD_SDFAT_LIBRARY```

    After interfacing the camera, microcontroller, and installing the drivers, it is time to test if everything works. First, connect the micro-USB to the Arduino Nano 33 BLE to power-on the microcontroller (it should automatically power-on when you plug the micro-USB). 

3. To get the source files for ArduCam 5MP camera, please copy "edge-impulse-final" to your Arduino Example folders: <br>

From now on, we will call the directory where you have cloned as `"<ROOT>"`.

In Mac OSx (where ever you have installed Arduino libraries):
 
 ``` 
     cp -r <ROOT>/edge-impulse-final  \
     /Users/<YOUR_USERNAME>/Documents/Arduino/libraries/Arduino_TensorFlowLite/examples/ 
 ```

In Windows 10:

```
   Xcopy /E <ROOT>\edge-impulse-final C:\<YOUR_USERNAME>\Documents\Arduino\libraries\Arduino_TensorFlowLite\examples
```


4. Compile the source files and upload the binary to the Arduino nano sense BLE. Before proceeding, please make sure the board and ports are configured correctly:<br>
    ``` Tools -> Board -> Arduino Nano 33 BLE ```
    ``` Tools -> Port -> COMx (Nano 33 BLE) ``` where COMx will depend upon your computer operating system. For Ubuntu, it is TTYxx. For OSx, it will be /dev/cu.usbmodem142xx(Arduino Nano 33 BLE)

5. Click the ``` Upload ``` button. 

6. Test that the application is working by opening the Serial monitor the log messages.

    ``` Tools -> Serial Monitor ```

7. In case if there are any issues with the camera interface, look for error messages such as:

    ```
    03:46:19.484 -> ReadData failed
    03:46:19.484 -> Image capture failed.
    ```
    The above error means the camera is not able to communicate with the Arduino nano BLE 33. Please make sure the wiring is correct and intact.<br>
