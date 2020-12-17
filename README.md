# Image Classification for Grocery Store Items: TinyML Applications in Retail
Final Project for CS249r: Tiny Machine Learning

### Folders in this Repository:

`edge-impulse-final`: contains the Arduino sketch necessary to deploy the model on an Arduino Nano 33 BLE using the Arducam 5MP Plus. Instructions on how to deploy this sketch are found below.

`dataset`: contains the 10 classes used in our research paper from the [Freiburg Grocery Dataset](https://github.com/PhilJd/freiburg_groceries_dataset).

`tflite`: contains the TFLite files from the models we explored in our paper (MobileNetV2, NN). Use the instructions at the bottom of this README to apply these TFLite files to your model.

`training`: contains screenshots of our work in Edge Impulse for each of the MobileNetV2 and NN models. Also contains Python notebooks with each model's neural network settings and code used to trained the models.

### NOTE: There are still some bugs in our system that need to be fixed, however the sketch is very close to being able to be deployed.
The following error currently results from deploying our model:
```
Arduino: 1.8.13 (Mac OS X), Board: "Arduino Nano 33 BLE"

Library Arduino_TensorFlowLite has been declared precompiled:
Using precompiled library in /Users/jessicaedwards/Documents/Arduino/libraries/Arduino_TensorFlowLite/src/cortex-m4/fpv4-sp-d16-softfp
/Users/jessicaedwards/Library/Arduino15/packages/arduino/hardware/mbed/1.3.0/variants/ARDUINO_NANO33BLE/linker_script.ld:138 cannot move location counter backwards (from 0000000020055f48 to 000000002003fc00)
collect2: error: ld returned 1 exit status
exit status 1
Error compiling for board Arduino Nano 33 BLE.
``` 
Given more time, we would be able to fix the model and calculate scores based on the likelihood of a detected object being in a certain class.

### Testing out the ArduCam interfacing with Arduino nano 33 BLE

Our model is similar to the Person Detection 249R assignment in interfacing the Arduino nano 33 BLE with the TinyML board. The instructions for setting up the camera and models are very similar to the instructions given in that assignment. 

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

### Deploying Other Models

Our project also includes the TFLite files of different models we trained using Edge Impulse and addressed in our paper. You can deploy these models by converting the TFLite file to a c++ source file to deploy it on the Arduino Nano 33 BLE. You can use xxd.exe available in git-bash for Windows 10. For Ubuntu, you can install it with sudo apt-get install -y xxd. For Mac OSx, xxd was already installed by default. The command to generate the c++ source file from the TFLite file is shown below:

Windows 10 (on git-bash) `$ xxd.exe -i <path to tflite file> > <name of the c++ source file>`

Ubuntu/Mac OSx: `$ xxd -i <path to tflite file> > <name of the c++ source file>`

This should convert the TFLite file of your trained model into a C++ source file that you can readily deploy in the microcontroller. This completes the exporting, freezing, and quantizing the trained model to TFLite, followed by converting it to C++ binary file. 

For deploying your trained model, you need to make sure you install Arduino TensorFlowLite 2.1.0-ALPHA. You will use the Arduino project as the basic template on which you will load your model data. Navigate to the `person_detect_model_data.cpp` file to copy your c++ binary data to store it in the variable alignas(8) const unsigned char `g_person_detect_model_data[]`.

Next navigate to the `edge-impulse-final.ino` file to change the kTensorArenaSize parameter. For determining the required size for your trained model, please use the utility designed by [Edge Impulse](https://github.com/edgeimpulse/tflite-find-arena-size). Using the correct kTensorArenaSize is important since if it is too small or too large, it will fail to initialize TFLite. Once you made these two important changes, you can compile and project and flash the binary to load onto the Arduino Nano 33 BLE using Arduino IDE/Platform I/O.
