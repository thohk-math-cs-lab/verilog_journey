
1. basic connection to module instance within a module (in a hierarchy)
```verilog
module top_module ( input a, input b, output out );
    // the code for mod_a is as below:
    //module mod_a ( input in1, input in2, output out );
    // <Module body>
	//endmodule
    mod_a instance1 ( .in1(a), .in2(b), .out(out));
    // connecting by "Name"
endmodule
```
![[Pasted image 20261009080701.png]]

2. (a) Variant 1: connnection by Positions

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);

    // module mod_a ( output, output, input, input, input, input );
	// we can connect by "Positions", by matching the order    
    mod_a instance1 (out1, out2, a, b, c, d);
endmodule
```

2. (b) Variant 2: connection of multiple inputs and outputs by Name

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);

    mod_a ( .out1(out1), .out2(out2), .in1(a), .in2(b), .in3(c), .in4(d) );
endmodule
```

3. module shift

![[Pasted image 20261009081925.png]]

```verilog
module top_module ( input clk, input d, output q );
    
    // Declare internal wires to connect the DFF chain
    wire q1; // connect from my_dff 1 to 2
    wire q2; // connect from my_dff 2 to 3
    
    // Connect the modules
    my_dff (.clk(clk), .d(d),  .q(q1));
    my_dff (.clk(clk), .d(q1), .q(q2));
    my_dff (.clk(clk), .d(q2), .q(q));
    
endmodule
```

