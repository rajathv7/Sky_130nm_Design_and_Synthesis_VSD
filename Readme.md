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
Consider a DFF with asynchronous reset and D input tied to VDD as shown below:

![sopt1](https://user-images.githubusercontent.com/88287721/166323083-cfe709c9-d0a9-4267-a001-dc3c8c665254.png)

Can the flop be replaced simply with an inverter with input as reset? The answer is <b>no</b> since the output goes high only on the rising edge of the clock and not on the falling edge of asynchronous reset. The proof of this analysis is shown in simulation and the output of synthesis in the figures below:

![sopt2](https://user-images.githubusercontent.com/88287721/166323716-eebc69bb-f9ae-4736-88ec-446414e1eff9.png)
![sopt3](https://user-images.githubusercontent.com/88287721/166323738-b405da76-80f0-48b8-9e70-9d9ff04f0c14.png)

However, instead of asynchronous reset; if the pin were asynchronous set, then the DFF can be replaced with a constant of 1 since there is no way the output of the DFF can become 0. The RTL code for this case and the synthesized output are shown below:

![sopt4](https://user-images.githubusercontent.com/88287721/166324220-68545afb-759f-49b5-9d8f-501f9a9fdc7b.png)
![sopt5](https://user-images.githubusercontent.com/88287721/166324235-4693209b-e0c8-4ba8-bbbf-b9d41675394a.png)

#### State optimization
#### Retiming
In this technique, the combinational logic between every pair of flops is made nearly equal so that the slack is more or less equally distributed between every pair of flops. It consists of moving the combinational around without affecting the overall functionality.

#### Sequential Logic Cloning
#### Sequential Optimization for Unused Outputs
Consider the following example of a 3-bit counter where only the LSB of the counter is given as output of the module, as shown below:

![oopt1](https://user-images.githubusercontent.com/88287721/166325332-4d9e993f-aec6-44ec-be46-ebd38c6256d9.png)

Since the higher two bits are unused, there will be removed in the optimization process. Since the LSB toggles on every clock cycle, the synthesized will have an inverter and only one DFF as shown below:

![oopt2](https://user-images.githubusercontent.com/88287721/166325567-c26c7197-47ee-42b2-93bf-25aa3d9460fc.png)

Notice that a second inverter is present just to invert the reset signal since the DFF of the library file has an active low reset, while we have coded an active high reset.

## Day 4: Gate Level Simulation (GLS) and Synthesis-Simulation Mismatch
### Gate Level Simulation (GLS)
GLS involves simulation of the netlist obtained after synthesis. Since the primary inputs and primary outputs of the netlist are same as that of the RTL, the same testbench used to test the RTL can be used to test the netlist as well. GLS is needed to verify the logical correctness of the design after synthesis and also for ensuring that the timing constraints of the design are met. 

GLS can be performed using iverilog like before, as shown below:

![gls1](https://user-images.githubusercontent.com/88287721/166326269-2205f3ea-29b8-481c-85fe-fb799581d93c.png)

But the Gate level verilog models need to be supplied to iverilog in addition to the netlist and testbench. 

### Synthesis-Simulation Mismatch
Synthesis-Simulation mismatch can occur due to <b>missing sensitivity list</b>, <b>blocking v/s non-blocking statements</b> and <b>non-standard Verilog coding</b>.

#### Missing Sensitivity List
Consider the following RTL of a 2:1 multiplexer:

![mis1](https://user-images.githubusercontent.com/88287721/166327468-544aaedf-38f8-4ea9-ae48-65ceda4c6ec0.png)

Notice that only <i>sel</i> is present in the sensitivity list. Therefore, the <i>always</i> block will not be executed if there is any change in <i>i0</i> or <i>i1</i>, thereby this exhibits a flop-like behaviour. But the synthesis step yields a 2:1 multiplexer since synthesis tools ignores the sensitivity list, as shown below:

![mis2](https://user-images.githubusercontent.com/88287721/166328123-5b05aebf-e1dc-40a4-867a-8e082cb4e831.png)

#### Blocking v/s Non-blocking Statements
Blocking statements in Verilog are indicated by the "=" assignment symbol. Blocking statements in an <b>always</b> block are executed in the order in which they appear (similiar to high level languages), and the execution of one statement happens only after the execution of the previous blocking statement is completed. Hence, the order in which blocking statements are written becomes very important. On the other hand, non-blocking statements are executed in parallel and hence the order in which they are written inside an <b>always</b> block does not matter. Once the <b>always</b> block is entered, the RHS of all the non-blocking statements are evaluated at once and assigned to the LHS variable at the indicated time instants.

Consider the following two cases where the order of the blocking statements are different:

![blo1](https://user-images.githubusercontent.com/88287721/166329388-8ef1867b-f5c4-406f-9f1b-3cd6650f9bc5.png)

The code on the left infers a flop since the previous value of q0 is to be used. Whereas the synthesis tool for both cases would generate combinational block only and not infer flops as shown below:

![blo2](https://user-images.githubusercontent.com/88287721/166329666-c77642f7-65bc-4ede-93cc-46c17906a676.png)

## Day 5: Optimization in Synthesis
### If construct
The <b>if</b> construct typically occurs in <b>always</b> block and gives a strict priority in the execution of the Verilog code, which may be represented as a nested set of multiplexers as shown below:

![if1](https://user-images.githubusercontent.com/88287721/166330019-edc0108f-e54b-4b89-ab28-73c026d22b8b.png)

#### Incomplete If construct
When the <b>else</b> portion of the <b>if</b> condition is not specified, it is called an incomplete if and leads to an inferred latch. This latch is undesirable most of the times in combinational circuits and can cause problems in timing closure. An example with illustration is shown below:

![nif1](https://user-images.githubusercontent.com/88287721/166330712-138623ca-c75a-43b6-892a-ab6a5563c529.png)

The following is the example of an RTL code where the definition of if is incomplete. The output is not specified when i0 takes the value 0. Hence, a latch is inferred as shown by the output of synthesis below:

![nif2](https://user-images.githubusercontent.com/88287721/166331215-0dcac9e7-fd54-4633-ba86-5be36ca9d5cc.png)
![nif3](https://user-images.githubusercontent.com/88287721/166331230-efca7624-4923-43ab-9f7b-fe37fddd8e9b.png)

### Case construct
<b>Case</b> construct also infers a multiplexer like the <b>if</b> statement, but the priorty present in <b>if</b> statement is absent here. Hence, if the variable inside the <b>case</b> matches with one of the options, the other options are also checked for a match. Hence, the designer must make sure to code the options in a non-overlapping manner.

#### Incomplete Case construct
Consider the following RTL where all the cases for the switching variable is not covered. Hence, latch will be inferred as in the incomplete if condition. 

![case1](https://user-images.githubusercontent.com/88287721/166331983-7ac242f6-74a6-4ab9-9dd9-f15815556512.png)
![case2](https://user-images.githubusercontent.com/88287721/166331998-381f6e51-a5a0-4ffa-9280-8a3c807517c6.png)

The solution to avoid the inferred latch is to use the <b>default</b> case and to specify the output in that case. The <b>default</b> case will be considered when none of the options match the switching variable. The Verilog code and the schematic of the completed case are shown below:

![case3](https://user-images.githubusercontent.com/88287721/166332495-2644b9b0-db5b-4ed1-bf88-8c5628918efd.png)
![case4](https://user-images.githubusercontent.com/88287721/166332510-cc702aea-eb09-46d8-885f-98c4c0efe865.png)

It must be noted however that merely by having a <b>default</b> case, the inferred latches may not be avoided. Latches will still be inferred if all the outputs are not specified in all the cases. For example, consider the following situation where the variable <i>x</i> is not defined in the case 2'b01. A latch will be inferred for the output <i>x</i>, as shown by the output of synthesis.

![case5](https://user-images.githubusercontent.com/88287721/166332976-19aebda1-bf45-43d6-b50f-85c2a86d4e64.png)
![case6](https://user-images.githubusercontent.com/88287721/166332986-03a6756a-8df8-4b64-9618-97e1d187f0b3.png)

### for loop
The <b>for</b> loop is used inside <b>always</b> for evaluation of expressions. The for loop comes in very handy when implementing very large multiplexers, de-multiplexers and decoders. The following RTL code implements an 4:1 multiplexer and can easily be modified to realize an 1024:1 multiplexer.

![for1](https://user-images.githubusercontent.com/88287721/166333587-38e66877-d2e0-4676-819e-df0f53ba226b.png)

### generate for loop
The <b>generate for</b> construct is used outside of <b>always</b> to instantiate hardware (typically multiple times). For example, a full-adder module may be instantiated multiple times to realize a ripple carry adder using the <b>generate for</b> construct as shown below.

![gen1](https://user-images.githubusercontent.com/88287721/166333975-a51a4e86-51fa-41f9-a8ed-ab41e47cda79.png)

### Acknowledgements and Gratitude
1. Kunal P Ghosh for organizing this workshop
2. Team Chipspirit
3. Shon Taware for support throughout the workshop
