
## Xiao ESP32S3 Sense with Edge Impulse for Keyword Spotting

This project demonstrates how to set up and run a keyword spotting model using the Xiao ESP32S3 Sense and Edge Impulse. The Xiao ESP32S3 Sense's powerful capabilities allow it to perform on-device machine learning, specifically keyword spotting, using data from its onboard microphone.

### Prerequisites

Before proceeding, ensure you have installed the following:
- **Arduino IDE** (latest version)
- **ESP32 Core**: Version **2.0.X** (Important: this code will **not work** with version 3.0.X or higher)
- **Edge Impulse CLI** for data collection and model deployment

### Installation

1. **Setting up the ESP32 Core in Arduino IDE**:
    - Open the Arduino IDE.
    - Navigate to **Tools > Board > Boards Manager**.
    - Search for **ESP32 by Espressif Systems**.
    - Install version **2.0.X** from the dropdown.
    - **Note**: This project will not function properly with version 3.0.X due to core changes in the libraries used.



### Code Adjustments

To ensure the code runs smoothly, you may need to adjust specific lines of code depending on your board setup and the ESP32 core version you're using.

#### Replace the following lines:

In `esp32_camera.ino`, you need to replace the following libraries with the ones compatible with the Xiao microphone :

```cpp

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#include "driver/i2s.h"
```
#### With these lines:


```cpp

#include<I2S.h>
#define SAMPLE_RATE 16000U
#define SAMPLE_BITS 16

```






#### Initialize the microphone in the void setup():

```cpp

    I2S.setAllPins(-1, 42, 41, -1, -1);
    if (!I2S.begin(PDM_MONO_MODE, SAMPLE_RATE, SAMPLE_BITS)) {
      Serial.println("Failed to initialize I2S!");
    while (1) ;
    }


```






#### Replace line 156 with these to properly read the audio:

```cpp

i2s_read((i2s_port_t)1, (void*)sampleBuffer, i2s_bytes_to_read, &bytes_read, 100);


```

#### With these lines

```cpp

  /* read data at once from i2s */
    esp_i2s::i2s_read(esp_i2s::I2S_NUM_0, (void*)sampleBuffer, i2s_bytes_to_read, &bytes_read, 100);

```





#### Replace function in line 244:

```cpp

static void microphone_inference_end(void)
{
    i2s_deinit();
    ei_free(inference.buffer);
}


```

#### With these lines

```cpp

static void microphone_inference_end(void)
{
    free(sampleBuffer);
    ei_free(inference.buffer);
});

```





#### Comment out deprecated code (line 251 to 295):

If you're using certain ESP32 core versions or configurations, you may need to comment out some lines that are no longer compatible or necessary:

```cpp


// static void microphone_inference_end(void)
// {
//     i2s_deinit();
//     ei_free(inference.buffer);
// }


// static int i2s_init(uint32_t sampling_rate) {
//   // Start listening for audio: MONO @ 8/16KHz
//   i2s_config_t i2s_config = {
//       .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_RX | I2S_MODE_TX),
//       .sample_rate = sampling_rate,
//       .bits_per_sample = (i2s_bits_per_sample_t)16,
//       .channel_format = I2S_CHANNEL_FMT_ONLY_RIGHT,
//       .communication_format = I2S_COMM_FORMAT_I2S,
//       .intr_alloc_flags = 0,
//       .dma_buf_count = 8,
//       .dma_buf_len = 512,
//       .use_apll = false,
//       .tx_desc_auto_clear = false,
//       .fixed_mclk = -1,
//   };
//   i2s_pin_config_t pin_config = {
//       .bck_io_num = 26,    // IIS_SCLK
//       .ws_io_num = 32,     // IIS_LCLK
//       .data_out_num = -1,  // IIS_DSIN
//       .data_in_num = 33,   // IIS_DOUT
//   };
//   esp_err_t ret = 0;

//   ret = i2s_driver_install((i2s_port_t)1, &i2s_config, 0, NULL);
//   if (ret != ESP_OK) {
//     ei_printf("Error in i2s_driver_install");
//   }

//   ret = i2s_set_pin((i2s_port_t)1, &pin_config);
//   if (ret != ESP_OK) {
//     ei_printf("Error in i2s_set_pin");
//   }

//   ret = i2s_zero_dma_buffer((i2s_port_t)1);
//   if (ret != ESP_OK) {
//     ei_printf("Error in initializing dma buffer with 0");
//   }

//   return int(ret);
// }

// static int i2s_deinit(void) {
//     i2s_driver_uninstall((i2s_port_t)1); //stop & destroy i2s driver
//     return 0;
// }>
```




### Running the Project

Once everything is set up, you can upload the code to your Xiao ESP32S3 Sense. Follow these steps:

1. Select your board as **XIAO ESP32S3 Sense**.
2. Compile and upload the code.

### Additional Notes

This project uses the onboard microphone of the Xiao ESP32S3 Sense for real-time audio capture. Edge Impulse’s platform is leveraged to train a model for detecting specific keywords, such as "yes" or "no". Once the model is deployed, the Xiao ESP32S3 Sense performs inference locally, triggering actions based on detected keywords.

### Troubleshooting

- If the device does not behave as expected, verify that you're using **ESP32 core version 2.0.X**. Higher versions have known issues with the libraries and configurations used in this project.
- Check the pin configurations for the microphone, and ensure the modifications listed above are correctly implemented.

### License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
