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

# Week 1 

<details>
	<summary>Day 1 - Introduction to Verilog RTL design and Synthesis </summary>
	
#  Day 1 - Introduction to Verilog RTL design and Synthesis
This section provides an introduction to Verilog RTL design and synthesis, covering the basics of the open-source simulator iverilog, practical labs using iverilog and gtkwave, an introduction to Yosys and logic synthesis, and hands-on labs with Yosys and Sky130 PDKs.


## Introduction to open-source simulator iverilog
Icarus Verilog is an open-source Verilog simulator that allows for the simulation of digital circuits described in the Verilog Hardware Description Language (HDL). It is a valuable tool for debugging and verifying the functionality of your designs before they are synthesized into hardware. This section will cover the basics of installing and using iverilog to compile and simulate your Verilog code.

## Labs using iverilog and gtkwave

### Lab1
In these labs, you will get hands-on experience with iverilog and the GTKWave waveform viewer. You will learn how to write simple Verilog modules, create testbenches to verify their functionality, and use GTKWave to visualize the simulation results. This will help you understand the behavior of your designs and debug any issues.

## Introduction to Yosys and Logic synthesis
Yosys is an open-source synthesis tool that converts your Verilog RTL code into a netlist, which is a description of the circuit in terms of logic gates. This section will introduce the fundamental concepts of logic synthesis, including how Yosys optimizes your design for area and performance.

## Labs using Yosys and Sky130 PDKs
These labs will guide you through the process of synthesizing your Verilog designs using Yosys and the Sky130 Process Design Kit (PDK). You will learn how to set up the synthesis flow, generate a netlist, and analyze the results. This will give you practical experience in preparing a design for fabrication.
</Details>


<Details>
<summary>Day 2 - Timing libs, hierarchical vs flat synthesis and efficient flop coding styles</summary>

# Day 2 - Timing libs, hierarchical vs flat synthesis and efficient flop coding styles
	This section focuses on timing libraries, the differences between hierarchical and flat synthesis approaches, and best practices for efficient flip-flop coding styles.

## Introduction to timing .libs
Timing libraries, or .lib files, are crucial for static timing analysis (STA). They contain detailed information about the timing characteristics of the standard cells used in your design. This section will explain the structure and content of these libraries and how they are used to verify that your design meets its timing requirements.

## Hierarchical vs Flat Synthesis
This section explores two different approaches to synthesis: hierarchical and flat. Hierarchical synthesis synthesizes each module of the design independently, which can be faster and easier to manage for large designs. Flat synthesis, on the other hand, synthesizes the entire design as a single unit, which can result in better optimization but may be more computationally intensive.

## Various Flop Coding Styles and optimization
The way you code your flip-flops in Verilog can have a significant impact on the performance and area of your synthesized circuit. This section will cover different coding styles for flip-flops and discuss how to write efficient and synthesizable code that meets your design goals.
</details>

<details>
<summary>Day 3 - Combinational and sequential optimizations</summary>
	
# Day 3 - Combinational and sequential optimizations

This section delves into the various optimization techniques used for both combinational and sequential logic during the synthesis process.

## Introduction to optimizations
Synthesis tools employ a wide range of optimization techniques to improve the quality of your design. This section provides an overview of these optimizations and explains how they help to reduce area, improve performance, and minimize power consumption.

## Combinational logic optimizations
This section focuses on optimization techniques specifically for combinational logic. Topics will include logic simplification, Boolean algebra, and other methods used to reduce the complexity of the circuit while preserving its functionality.

## Sequential logic optimizations
This section covers optimization techniques for sequential logic, such as state machine encoding, retiming, and clock gating. These techniques are used to improve the timing and power characteristics of your sequential circuits.

## Sequential optimizations for unused outputs
In some cases, the outputs of sequential elements may not be used by any other part of the design. This section will discuss how synthesis tools can identify and remove these unused outputs to reduce the overall area of the circuit.
</details>

<details>
	
<summary> Day 4 - GLS, blocking vs non-blocking and Synthesis-Simulation mismatch </summary>
	
# Day 4 - GLS, blocking vs non-blocking and Synthesis-Simulation mismatch
This section covers Gate-Level Simulation (GLS), the important distinction between blocking and non-blocking assignments, and the potential for mismatches between synthesis and simulation results.

## GLS, Synthesis-Simulation mismatch and Blocking/Non-blocking statements
Gate-Level Simulation (GLS) is a type of simulation that is performed on the synthesized netlist. It is used to verify the functionality and timing of the design after it has been optimized by the synthesis tool. This section will also discuss the critical difference between blocking (=) and non-blocking (<=) assignments in Verilog and how they can lead to synthesis-simulation mismatches if not used correctly.

## Labs on GLS and Synthesis-Simulation Mismatch
In these labs, you will perform Gate-Level Simulations on your synthesized designs and learn how to identify and debug synthesis-simulation mismatches. This will give you a deeper understanding of the importance of writing synthesizable Verilog code.

## Labs on synth-sim mismatch for blocking statement
These labs will focus specifically on how the use of blocking statements can lead to synthesis-simulation mismatches. You will see practical examples of this issue and learn how to avoid it in your own designs.
</details>

<details>

<summary>Day 5 - Optimization in synthesis</summary>
	
# Day 5 - Optimization in synthesis
This section explores advanced optimization techniques used in synthesis, focusing on control structures like if and case statements, as well as for loops and generate blocks.

## If Case constructs
This section will discuss how synthesis tools handle if and case statements in Verilog. You will learn how these constructs are translated into logic gates and how different coding styles can affect the quality of the synthesized circuit.

## Labs on "Incomplete If Case"
In these labs, you will learn about the concept of "incomplete if case" statements and how they can lead to the inference of latches in your design. You will see how to identify and fix these issues to ensure that your design is purely combinational.

## Labs on "Incomplete overlapping Case"
These labs will cover the topic of "incomplete overlapping case" statements. You will learn how these constructs can lead to unexpected behavior in your synthesized circuit and how to write your code to avoid these issues.

## for loop and for generate
This section will explore the use of for loops and generate blocks in Verilog. You will learn how these constructs can be used to create regular and scalable hardware structures and how they are handled by the synthesis tool.

## Labs on "for loop" and "for generate"
In these labs, you will get hands-on experience with for loops and generate blocks. You will learn how to use these constructs to create complex hardware designs and how to write efficient and synthesizable code.
</details>
