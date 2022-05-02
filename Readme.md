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

An illustration of synthesis of a flopped 2:1 multiplexer is shown below:
![synthesis_illust](https://user-images.githubusercontent.com/88287721/166296494-78cced46-7b6b-43fd-976a-a019e0ff070e.png)

#### Lab Example:
The 2:1 multiplexer shown above is synthesized now by invoking yosys as follows:
![mux_syn1](https://user-images.githubusercontent.com/88287721/166297267-dd0e6b8c-f7a8-4bb3-a4ef-3020db866d20.png)
![mux_syn2](https://user-images.githubusercontent.com/88287721/166297298-edafb9e5-743d-4934-b71c-4da9d0fcec82.png)
![mux_syn3](https://user-images.githubusercontent.com/88287721/166297321-d0d95d24-8ec6-49e6-9d75-2e70aca72702.png)
Clearly, the result of synthesis is a 2:1 multiplexer as expected.

## Day 2: Timing libs, hierarchical v/s flat synthesis and efficient flop coding styles
### Timing libs
The library file which we are using is called <b>sky_130_fd_sc_hd__tt_025C_1v80.lib</b>. Here <b>tt</b> means <i>typical, typical</i> process corner; meaning that both NMOS and PMOS are of typical speed, <b>025C</b> indicates the temperature to be 25^\circ C and <b>1v80</b> indicates that the supply voltage is 1.8 V. This is important to note because the performance of digital (as well as analog) circuits varies in accordance with process, voltage and temperature (PVT) variations. A snapshot of the library file is shown below:
![lib1](https://user-images.githubusercontent.com/88287721/166298873-245fb947-7291-4cd6-8a0f-bec4270325f3.png)
We can observe the units of various parameters of the design, such as Volt for Voltage, milli-Ampere for current, nanosecond for delay etc. and the delay model is table lookup.
![lib2](https://user-images.githubusercontent.com/88287721/166298896-47d65352-e413-40c8-b55f-caf2adf85d8d.png)
![lib3](https://user-images.githubusercontent.com/88287721/166298916-75c50d2d-4168-47c2-90e0-a01bd204a779.png)
Here, we can see the various gates present in the library. For example, the gate named a2111o_1 is shown along with the leakage powers for different combinations of the inputs. The definition of the gate can also be seen in the second figure as <i>X = (A1 & A2) | B1 | C1 | D1</i>.

Now let us look at three flavours of the 2-input AND gate as shown below, namely <i>and2_0</i>, <i>and2_2</i> and <i>and2_4</i>.
![lib4](https://user-images.githubusercontent.com/88287721/166303866-845c783c-8289-4cbd-bee3-1a54e8c7768e.png)
The figure clearly shows that area-wise and power-wise, <i>and2_4</i> is the largest while <i>and2_0</i> is the smallest. On the contrary delay-wise, <i>and2_0</i> will be the largest while <i>and2_4</i> will be the smallest.

### Hierarchical Synthesis
Now, let us consider a top module comprising of two sub-modules as shown below:
![hier_rtl](https://user-images.githubusercontent.com/88287721/166304631-d6847e3b-985c-403f-9819-76b3261f94d9.png)
![hier_fig](https://user-images.githubusercontent.com/88287721/166304611-c3297fe1-ffe3-4394-9b4c-0ffc7ff05079.png)

Let us synthesize this module and observe the results.
![hier1](https://user-images.githubusercontent.com/88287721/166308765-d81eaa15-a66c-40b0-ad69-716de85531c5.png)
![hier2](https://user-images.githubusercontent.com/88287721/166308774-8555cbe8-f19d-4c2c-9bdb-e15caa93e368.png)
![hier3](https://user-images.githubusercontent.com/88287721/166308785-721076c0-1ad2-4569-81c1-780a205a61fe.png)
![hier4](https://user-images.githubusercontent.com/88287721/166308793-7458ebb2-cbb3-4e62-9c13-b09c5d3d144e.png)
Clearly, two modules are instantiated in the synthesis step.

The netlist is obtained as follows:

![hier5](https://user-images.githubusercontent.com/88287721/166309414-c6c191c8-14ed-4e2e-847f-63fb9ebd3a8c.png)

By using the <i>flatten</i> command at the yosys prompt, the top module can be reduced to a single file with both the modules instantiated in the same file as shown below:

![flat1](https://user-images.githubusercontent.com/88287721/166310276-a0ad0555-e9a6-43d9-b38a-d1f7e6a6a605.png)

The block diagram of the synthesis step is shown below:

![flat2](https://user-images.githubusercontent.com/88287721/166310510-534c56bd-7be7-4bbb-beda-5eb8e9e7de76.png)

The two sub-modules can be synthesized individually. Sub-module1 is synthesized and is shown as follows:

![flat3](https://user-images.githubusercontent.com/88287721/166310917-6c284697-478a-4de8-b609-11c077aee1dc.png)

Synthesizing sub-modules separately can be useful if the sub-module is instantiated multiple number of times; or if the top-module is very big and the tool is not able to handle synthesizing the whole of it at once (divie-and-conquer approach).

### Flop Coding Styles
Digital design is typically done using flops to avoid excessive glitching in combinational circuits. The output of the flops drives the input of combinational logic, while the output of combinational logic is captured by the flops. Hence, it becomes essential to initialize the flops through the set/reset pin. The different flavours of flop designs are asynchronous reset/set and synchronous reset/set. 

The Verilog code of a flop with asynchronous reset and the corresponding synthesized output is shown below:

![async_res1](https://user-images.githubusercontent.com/88287721/166312848-9224a7c7-6e55-4ad5-bf89-7113529d0120.png)
![async_res2](https://user-images.githubusercontent.com/88287721/166312967-a6b8ac9f-3e00-4a66-9319-29e3e37d07a5.png)

Since the D-flip-flops can potentially be located in a different technology library, we need to inform yosys of its location. Hence, if the design involves flops, then the following line needs to be added before converting RTL to gates:

]$ dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

### Interesting Combinational Optimizations
Suppose we are implementing a block which multiplies the 3-bit input by a factor of 2 to yield a 4-bit output, as shown below:

![opt1](https://user-images.githubusercontent.com/88287721/166314046-be81b26d-c38f-488f-ae9e-68fa2ee5bbd4.png)

Since it can be clearly seen that multiplying by 2 is equivalent to shift left by one bit, it turns out the above multiplier does not need any gates to be realized as confirmed by the netlist shown below:

![opt2](https://user-images.githubusercontent.com/88287721/166314327-d3b1952c-40d9-412d-bbf1-df7bb9a006bd.png)

Similarly, if we are implementing a block which multiplies the 3-bit input by a factor of 9 to yield a 6-bit output, as shown below:

![opt3](https://user-images.githubusercontent.com/88287721/166314708-ea2db62c-a197-4e91-bd31-1482dc41d9f9.png)

It is easy to see again that the output is the concatenation of the input twice, as shown by the synthesis tool below:

![opt4](https://user-images.githubusercontent.com/88287721/166314822-4d3a8971-2e76-4152-862d-b767f9d539e5.png)

The same is confirmed by writing out the netlist, as shown below:

![opt5](https://user-images.githubusercontent.com/88287721/166315057-964d4643-aadd-457a-b6c6-58f908f8f115.png)

## Day 3: Combinational and sequential optmizations
### Combinational Optimization
Combinational optimization is to be performed to obtain the best design in terms of area and power, subject to a certain performace requirement. The two techniques of combinational optimization are <b>constant propagation</b> and <b>Boolean logic optimization</b>.
  
#### Constant Propagation
Consider the following digital circuit in which A is tied to ground. Then the circuit just simplies to an inverter. With this simplification, the number of transistors required to realize this logic comes down from 6 to just 2.

![copt1](https://user-images.githubusercontent.com/88287721/166316227-8c03b9d0-0994-4be7-9603-1f9573fe9fd4.png)

#### Boolean optimization
Consider the following example with the nested ternary operator. By applying Boolean laws, the complicated expression is just reduced to a single gate as shown below:

![copt2](https://user-images.githubusercontent.com/88287721/166316597-1b621b9f-36b3-4d02-b6cb-0bf823641182.png)

Consider the following example where a ternary operator is used, but it simplifies to an <b>and</b> gate with Boolean laws.
![copt3](https://user-images.githubusercontent.com/88287721/166318884-ffa41d5e-5d00-41ef-a25b-ed5da9e1b716.png)

The output of synthesis after running the command <i>opt_clean -purge</i> is as follows, consisting of just an <b>and</b> gate as expected:

![copt4](https://user-images.githubusercontent.com/88287721/166319125-c94b0f23-d837-4805-9a48-1fdef5140421.png)

### Sequential Optimization
#### Sequential Constant Propagation

#### State optimization
#### Retiming
#### Sequential Logic Cloning

## Day 4: Gate Level Simulation (GLS) and Simulation-Synthesis Mismatch
### Gate Level Simulation
### Blocking and Non-blocking Statements

## Day 5: Optimization in Synthesis
### If construct
#### Incomplete If construct
### Case construct
#### Incomplete Case construct
### for loop
### generate for loop







