# Design and Synthesis using Skywater 130 nm

This is a journal for the Workshop on Design and Synthesis using Sky water 130nm Technological Library offered from 27th April to 1st May 2022.
![ad](https://user-images.githubusercontent.com/88287721/166251091-ac2e583d-36c4-4074-8b14-24bb4fbcfb5b.png)

## Day1: Introduction to Verilog RTL Design and Synthesis
### Open Source Simulator: iverilog and waveform viewer gtkwave

Once the RTL design along with the test bench is ready, iverilog can be invoked with the following command:

]$ iverilog <design.v> <test_bench.v>

For example, if the RTL design file is named good_mux.v and the testbench is called tb_good_mux.v, then the simulator is invoked using the following command:

]$ iverilog good_mux.v tb_good_mux.v
A snap shot of the RTL design and testbench of good_mux is shown below:
![good_mux_code](https://user-images.githubusercontent.com/88287721/166254702-8b0bf1c1-2a8b-4d8b-b65e-68165f872424.png)
Note that the testbench cannot have any primary inputs or outputs in its top module and that the design block is instantiated under the name <i>uut</i> (unit under test).
The order of the design file and the testbench file does not matter as long as both are present. As a result of the execution of iverilog, a file by name "a.out" is dumped in the working directory, which can be executed by typing out ]$ ./a.out which then results in a <i>value change dump</i> (.vcd) file, in this case, called tb_good_mux.vcd. The value change dump file can be viewed graphically using the freeware tool GTKwave by running the command ]$ gtkwave tb_good_mux.vcd

The waveform shown by GTKwave is as follows:
![gtk_good_mux1](https://user-images.githubusercontent.com/88287721/166253869-d1b11b4a-4c55-48c3-8140-723c801c627c.png)
It is easy to see that the behaviour is as expected from a 2:1 multipler.

### Open Source Synthesis Tool: yosys
Synthesis is the process of converting the RTL code into gate level netlist, given a technology library contains the various gates and optional design constraints. Hence the input to a synthesis tool is the RTL code and the technology library (and also design constraints optionally) and the output is the gate-level netlist. This can be pictorially shown as follows:
![yosys_setup](https://user-images.githubusercontent.com/88287721/166255649-bb166b27-f1c4-4e54-a8dd-b58068c7e986.png)

Once the netlist is obtained from synthesis step, the netlist must be subjected to a gate level simulation (GLS) to verify the correctness of the synthesized netlist. Since the primary inputs and primary outputs of the netlist must be the same as that of the RTL design, the same testbench developed to functionally verify the RTL design can be used for gate level simulation also. In the same manner as described above, iverilog can be used for simulation and gtkwave to view the results. These steps can be shown pictorially as follows:
![verify_synthesis](https://user-images.githubusercontent.com/88287721/166256560-cc5cbf04-78b4-490b-8464-d8315eceb605.png)

The .lib file consists of all logic gates (such as inverter, AND gate, OR gate, etc.) and different flavours of the said gates such as varying number of inputs (for  example, 2-input AND gate, 3-input AND gate, 4-input AND gate etc.) and of varying transistor sizes so as to yield different delays (slow, medium, fast). 

Why should the library consist of different flavours of the gates?
In digital design employing edge-triggered flip-flops as shown below; the clock-to-Q delay of flop A, the combinational delay and the setup time of flop B has to be completed within one clock period. Therefore, to ensure small clock periods (higher frequencies of operation), it is desirable to have gates having less delays.
![setup_time](https://user-images.githubusercontent.com/88287721/166277128-e3e19092-4994-4869-80c4-095080694cfb.png)
On the other hand though, the output of the flop has to stay stable for a certain <i>hold time</i> after the arrival of the clock edge. This calls for a minimum delay of the combinational logic. Hence gates of larger delays could be used to meet the hold time constraint.

The digital circuits typically need to drive loads which are capacitive in nature. Hence if more current can be sourced to the load, the delay will be lesser. Wider transistors can source more current, thereby giving lesser delay; but they come at the cost of increased area and increased leakage power. In summary, wider transistors give smaller delay at the cost of increased area and power; while narrow transistors give larger delay consuming lesser area and lesser power. This is the reason for the technology library to consist of gates of different flavours. The synthesis tool needs to be guided therefore by the designer by providing design constraints so that the synthesis tool can select the appropriate flavour of the gate required to satisy the design specifications.

#### Lab Example:





