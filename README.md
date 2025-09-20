# sfal-vsd-summary

<details>
	<summary>Day 0 - Tools Installation </summary>
	
# Day 0 - Tools Installation
## Yosys
A framework for Verilog RTL synthesis.
```
# Update package lists
sudo apt-get update

# Clone the Yosys repository
git clone [https://github.com/YosysHQ/yosys.git](https://github.com/YosysHQ/yosys.git)
cd yosys

# Install dependencies
sudo apt-get install build-essential clang bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev zlib1g-dev

# Build and install
make config-gcc
make
sudo make install
```
## Icarus Verilog (iverilog)
A Verilog simulation and synthesis tool.
```
sudo apt-get update
sudo apt-get install iverilog
```
## GTKWave
A fully featured GTK+ based waveform viewer.
```
sudo apt-get update
sudo apt-get install gtkwave
```
## ngspice
A mixed-level/mixed-signal circuit simulator.
```
# First, download the tarball from [https://sourceforge.net/projects/ngspice/files/](https://sourceforge.net/projects/ngspice/files/)
# Then, run the following commands, replacing 'ngspice-XX' with the correct version number.
tar -zxvf ngspice-XX.tar.gz
cd ngspice-XX
mkdir release
cd release
../configure --with-x --with-readline=yes --disable-debug
make
sudo make install
```
## Magic
A VLSI layout tool.
```
# Install all dependencies at once
sudo apt-get install m4 tcsh csh libx11-dev tcl-dev tk-dev \
    libcairo2-dev mesa-common-dev libglu1-mesa-dev libncurses-dev

# Clone the repository
git clone [https://github.com/RTimothyEdwards/magic.git](https://github.com/RTimothyEdwards/magic.git)
cd magic

# Build and install
./configure
make
sudo make install
```
## OpenLANE
An automated RTL to GDSII flow that runs in a Docker environment.
```
1. Install Dependencies
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
```
## 2. Install Docker
```
# Add Docker's official GPG key and set up the repository
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
## 3. Manage Docker Permissions
```
# Add your user to the 'docker' group to run commands without sudo
sudo groupadd docker
sudo usermod -aG docker $USER

# IMPORTANT: YOU MUST REBOOT YOUR SYSTEM NOW FOR THIS TO TAKE EFFECT
sudo reboot

# After rebooting, verify by running: docker run hello-world
```
## 4. Install OpenLANE
```
# Navigate to your home directory and clone the repository
cd $HOME
git clone [https://github.com/The-OpenROAD-Project/OpenLane.git](https://github.com/The-OpenROAD-Project/OpenLane.git)

# Go into the directory and build the environment
cd OpenLane
make

# Run the test set to ensure everything works
make test
```
</details>

<details>
<summary> Day 0 - The Digital Chip Design and Verification Flow </summary>
  
# Day 0 - The Digital Chip Design and Verification Flow

This document outlines the standard process for taking an application from a high-level specification to a physical, manufactured microchip. The core principle is to create and verify the design at different levels of abstraction, ensuring the output remains consistent at every stage.

The ultimate goal is to ensure the final silicon chip's output (**O4**) matches the outputs from all previous stages of design and simulation (**O3**, **O2**, and **O1**).

`O4 == O3 == O2 == O1`



---

##  Stage 1: High-Level Modeling (Output O1)

Before designing any hardware, we first model the chip's intended application in a high-level language like C or C++. This model serves as a "golden reference" to ensure the logic is correct and meets the specification.

* **Goal:** Create a functional C model of the application's specification.
* **Process:**
    1.  The application logic is written in C.
    2.  A testbench, also in C, is created to provide inputs to the model and check its output.
    3.  The code is compiled using a standard C compiler (like GCC).
* **Verification:** The output from this stage is **O1**. We run the testbench to confirm that **O1** matches the expected result. This validates the core algorithm.

---

##  Stage 2: RTL Design (Output O2)

Once the high-level logic is confirmed, we create a "soft copy" of the hardware using a Hardware Description Language (HDL).

* **Goal:** Describe the hardware's behavior and structure in code.
* **Process:** The hardware design, including the processor and peripherals, is written in an HDL like Verilog or SystemVerilog. This is called the Register-Transfer Level (RTL) design.
* **Verification:** We run the same application from Stage 1 on a simulation of the RTL hardware. The output of this simulation is **O2**. The critical check here is to ensure `O2 == O1`. This proves that our hardware design correctly implements the application's logic.

---

##  Stage 3: Synthesis & SoC Integration (Output O3)

In this stage, the abstract RTL code is converted into a design made of actual logic gates, and all the chip's components are connected.

* **Goal:** Convert the RTL design into a gate-level netlist and integrate all components into a full System on a Chip (SoC).
* **Process:**
    1.  **Synthesis:** The RTL code is fed into a synthesis tool, which converts it into a **Gate Level Netlist**—a description using standard logic gates (AND, OR, etc.).
    2.  **Component Integration:** The synthesized netlist is combined with other essential blocks like **Macros** (reusable blocks like clock dividers) and **Analog IPs** (ADCs, PLLs).
    3.  **SoC Assembly:** All blocks are connected with General Purpose Input/Output (GPIOs) to create the complete SoC design.
* **Verification:** The application is run on a simulation of this final, integrated gate-level design. The output is **O3**. We must verify that `O3 == O2 == O1`.

---

##  Stage 4: Physical Design & Tapeout (GDSII)

This is where the digital design is transformed into a physical layout—a detailed blueprint for manufacturing.

* **Goal:** Create the final manufacturing file (**GDSII**).
* **Process:**
    1.  **Physical Design:** This involves **floorplanning** (arranging major blocks), **placement** (placing logic gates), and **routing** (drawing the metal wires to connect everything).
    2.  **GDSII Generation:** The final layout is saved in a GDSII file format, which is the blueprint sent to the factory (foundry).
    3.  **Final Checks:** The GDSII file undergoes rigorous checks like **DRC** (Design Rule Check) and **LVS** (Layout vs. Schematic).
* **Tapeout:** The process of sending the final, verified GDSII file to the manufacturing plant.

---

##  Stage 5: Chip Validation (Output O4)

After manufacturing, the physical chip is returned from the foundry. This is often called "Tape-in" or "Silicon Bring-up."

* **Goal:** Test the real silicon chip to ensure it works correctly.
* **Process:** The physical chip is placed on a test board, and the original C testbench is used to feed it inputs and measure its outputs.
* **Verification:** The output from the physical chip is **O4**. The final, ultimate verification is confirming that `O4 == O3 == O2 == O1`. If this holds true, the chip is ready for the market.

---

## Illustrative C Code Example (Stage 1)

Here is a simple example demonstrating Stage 1: creating a C model for a Multiply-Accumulate (MAC) application and testing it with a testbench.

### 1. `application_model.h` - The Header File
This file declares the function that our application model provides.
```c
#ifndef APPLICATION_MODEL_H
#define APPLICATION_MODEL_H

// This function represents the core logic of our application
int run_mac_operation(int a, int b, int c);

#endif // APPLICATION_MODEL_H
```
#### 2. `application_model.c` - The C Model
This file contains the actual C implementation of our application, which serves as the golden reference.
```
#include "application_model.h"

// Implementation of the MAC operation
// This is the "specification" or "golden reference" model
int run_mac_operation(int a, int b, int c) {
    int product = a * b;
    int result = product + c;
    return result;
}
```
### 3. `testbench.c` - The Testbench
This file tests our C model. It provides inputs, gets the output (O1), and compares it to a known correct answer.
```
#include <stdio.h>
#include "application_model.h"

int main() {
    // 1. Define test inputs
    int input_a = 10;
    int input_b = 5;
    int input_c = 20;

    // 2. Define the expected "golden" output for these inputs
    // Expected = (10 * 5) + 20 = 70
    int expected_o1 = 70;

    printf("--- C Testbench Running ---\n");
    printf("Inputs: a=%d, b=%d, c=%d\n", input_a, input_b, input_c);
    printf("Expected Output (O1): %d\n", expected_o1);

    // 3. Run the application model to get the actual output
    int actual_o1 = run_mac_operation(input_a, input_b, input_c);
    printf("Actual Output (O1) from C Model: %d\n", actual_o1);

    // 4. Verification Step: Compare actual output with expected output
    if (actual_o1 == expected_o1) {
        printf("VERIFICATION PASSED: actual_o1 == expected_o1\n");
    } else {
        printf("VERIFICATION FAILED: actual_o1 != expected_o1\n");
    }

    return 0;
}
```
### 4. `Makefile` - How to Compile
This file tells the `gcc` compiler how to build the final executable program.
```
# Makefile to compile the testbench and application model

# Compiler
CC = gcc

# Compiler flags
CFLAGS = -Wall -Werror

# Target executable name
TARGET = testbench

# Source files
SOURCES = testbench.c application_model.c

# Default rule to build the target
all: $(TARGET)

$(TARGET): $(SOURCES)
	$(CC) $(CFLAGS) -o $(TARGET) $(SOURCES)

# Rule to clean up generated files
clean:
	rm -f $(TARGET)
```
How to Run This Example
1. Save the four code blocks above into their respective files (`application_model.h`, `application_model.c`, `testbench.c`, `Makefile`).

2. Open a terminal in that directory.

3. Compile the code by running the `make` command:
```
make
```
### 4.Execute the compiled program:
```
./testbench
```
#### 5.You will see the following output, which confirms that the C model passed the test.
```
--- C Testbench Running ---
Inputs: a=10, b=5, c=20
Expected Output (O1): 70
Actual Output (O1) from C Model: 70
VERIFICATION PASSED: actual_o1 == expected_o1
```
</details>

