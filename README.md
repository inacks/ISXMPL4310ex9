# IS4310 Modbus STM32 Code Example

This STM32 CubeIDE project communicates with the IS4310 Modbus RTU chip via I2C.

It demonstrates:
* How to read a potentiometer (simulating a sensor) and store its state in **Holding Register 0**.
* How to control an RGB LED (simulating an actuator) using GPIO pins based on values in **Holding Registers 1, 2, and 3**.

The following code is an abstraction of the 'main.c' omitting the HAL libraries. Check the 'rar' file for the full STM32 project.

'''cpp
uint16_t readHoldingRegister(uint16_t registerAdressToRead) {
	  uint8_t IS4310_I2C_Chip_Address; // This variable stores the I2C chip address of the IS4310.
	  IS4310_I2C_Chip_Address = 0x11; // The IS4310's I2C address is 0x11.
	  // The STM32 HAL I2C library requires the I2C address to be shifted left by one bit.
	  // Let's shift the IS4310 I2C address accordingly:
	  IS4310_I2C_Chip_Address = IS4310_I2C_Chip_Address << 1;

	  // The following array will store the read data.
	  // Since each holding register is 16 bits long, reading one register requires reading 2 bytes.
	  uint8_t readResultArray[2];

	  // This variable will contain the final result:
	  uint16_t readResult;


	  /*
	   * This is the HAL function to read from an I2C memory device. The IS4310 is designed to operate as an I2C memory.
	   *
	   * HAL_I2C_Mem_Read parameters explained:
	   * 1. &hi2c1: This is the name of the I2C that you're using. You set this in the CubeMX. Don't forget the '&'.
	   * 2. IS4310_I2C_Chip_Address: The I2C address of the IS4310 (must be left-shifted).
	   * 3. registerAdressToRead: The holding register address to read from the IS4310.
	   * 4. I2C_MEMADD_SIZE_16BIT: You must indicate the memory addressing size. The IS4310 memory addressing is 16-bits.
	   * 	This keyword is an internal constant of HAL libraries. Just write it.
	   * 5. readResultArray: An 8-bit array where the HAL stores the read data.
	   * 6. 2: The number of bytes to read. Since one holding register is 16 bits, we need to read 2 bytes.
	   * 7. 1000: Timeout in milliseconds. If the HAL fails to read within this time, it will skip the operation
	   *    to prevent the code from getting stuck.
	   */
	  HAL_I2C_Mem_Read(&hi2c1, IS4310_I2C_Chip_Address, registerAdressToRead, I2C_MEMADD_SIZE_16BIT, readResultArray, 2, 1000);

	  // Combine two bytes into a 16-bit result:
	  readResult = readResultArray[0];
	  readResult = readResult << 8;
	  readResult = readResult | readResultArray[1];

	  return readResult;
}

void writeHoldingRegister(uint16_t registerAdressToWrite, uint16_t value) {
	uint8_t IS4310_I2C_Chip_Address;  // I2C address of IS4310 chip (7-bit).
	IS4310_I2C_Chip_Address = 0x11; // IS4310 I2C address is 0x11 (7-bit).
	// STM32 HAL expects 8-bit address, so shift left by 1:
	IS4310_I2C_Chip_Address = IS4310_I2C_Chip_Address << 1;

	// The HAL library to write I2C memories needs the data to be in a uint8_t array.
	// So, lets put our uint16_t data into a 2 registers uint8_t array.
	uint8_t writeValuesArray[2];
	writeValuesArray[0] = (uint8_t) (value >> 8);
	writeValuesArray[1] = (uint8_t) value;

	/*
	 * This is the HAL function to write to an I2C memory device. To be simple and easy to use, the IS4310 is designed to operate as an I2C memory.
	 *
	 * HAL_I2C_Mem_Write parameters explained:
	 * 1. &hi2c1: This is the name of the I2C that you're using. You set this in the CubeMX. Don't forget the '&'.
	 * 2. IS4310_I2C_Chip_Address: The I2C address of the IS4310 (must be left-shifted).
	 * 3. registerAdressToWrite: The holding register address of the IS4310 we want to write to.
	 * 4. I2C_MEMADD_SIZE_16BIT: You must indicate the memory addressing size. The IS4310 memory addressing is 16-bits.
	 * 	This keyword is an internal constant of HAL libraries. Just write it.
	 * 5. writeValuesArray: An 8-bit array where we store the data to be written by the HAL function.
	 * 6. 2: The number of bytes to write. Since one holding register is 16 bits, we need to write 2 bytes.
	 * 7. 1000: Timeout in milliseconds. If the HAL fails to write within this time, it will skip the operation
	 *    to prevent the code from getting stuck.
	 */
	HAL_I2C_Mem_Write(&hi2c1, IS4310_I2C_Chip_Address, registerAdressToWrite, I2C_MEMADD_SIZE_16BIT, writeValuesArray, 2, 1000);
}

while (1)
  {
	// This will sotre the potentiometer value:
	uint16_t potentiometerValue;
	// This will store the read value of the Holding Registers 1, 2 and 3:
	uint16_t holdingRegister1;
	uint16_t holdingRegister2;
	uint16_t holdingRegister3;

	// Read Holding Registers 1, 2 and 3:
	holdingRegister1 = readHoldingRegister(1);
	holdingRegister2 = readHoldingRegister(2);
	holdingRegister3 = readHoldingRegister(3);

	// If the value of each read Holding register is diferent from 0,
	// let's turn on the corresponding LED:
	if (holdingRegister1 >= 1) {
	  HAL_GPIO_WritePin(RGB_Red_GPIO_Port, RGB_Red_Pin, GPIO_PIN_SET);
	}
	else {
	  HAL_GPIO_WritePin(RGB_Red_GPIO_Port, RGB_Red_Pin, GPIO_PIN_RESET);
	}

	if (holdingRegister2 >= 1) {
	  HAL_GPIO_WritePin(RGB_Green_GPIO_Port, RGB_Green_Pin, GPIO_PIN_SET);
	}
	else {
	  HAL_GPIO_WritePin(RGB_Green_GPIO_Port, RGB_Green_Pin, GPIO_PIN_RESET);
	}

	if (holdingRegister3 >= 1) {
	  HAL_GPIO_WritePin(RGB_Blue_GPIO_Port, RGB_Blue_Pin, GPIO_PIN_SET);
	}
	else {
	  HAL_GPIO_WritePin(RGB_Blue_GPIO_Port, RGB_Blue_Pin, GPIO_PIN_RESET);
	}


	/*
	 * Read ADC value from potentiometer (0-4095),
	 * and write it to Holding Register 0.
	 */
	HAL_ADC_Start(&hadc1); // Start the HAL ADC
	HAL_ADC_PollForConversion(&hadc1, 400); // Perform a ADC read
	// Get the ADC value:
	potentiometerValue = HAL_ADC_GetValue(&hadc1);
	// Store the ADC value to the Holding Register 0:
	writeHoldingRegister(0, potentiometerValue);
	// Stop the HAL ADC
	HAL_ADC_Stop(&hadc1);

}
'''

💡 Test this example using the **Kappa4310Rasp Evaluation Board**:
👉 [https://www.inacks.com/kappa4310rasp](https://www.inacks.com/kappa4310rasp)

And the **Nucleo-C071**: [https://www.st.com/en/evaluation-tools/nucleo-c071rb.html](https://www.st.com/en/evaluation-tools/nucleo-c071rb.html)



📄 Download the **IS4310 datasheet**:
👉 [https://www.inacks.com/is4310](https://www.inacks.com/is4310)

For more information visit [www.inacks.com](https://www.inacks.com/is4310)
