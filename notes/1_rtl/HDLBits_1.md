### Wire

1. Simple wire
```verilog
module top_module( input in, output out );
	assign out = in;
	// directional - from in to out
endmodule
```

2. multiple wires
```verilog
module top_module( 
    input a,b,c,
    output w,x,y,z );
	
    assign w = a;
    assign x = b; // b is input for both x
    assign y = b; // and y
    assign z = c;
    // If we're certain about the width of each signal, using 
	// the concatenation operator is equivalent and shorter:
	// assign {w,x,y,z} = {a,b,b,c};
	
endmodule
```

3. negation
```verilog
module top_module( input in, output out );
	assign out = !in; // ! is logical negation
					  // ~ is bitwise negation,
					  // can also work here for SINGLE bit
endmodule
```

4. AND gate
```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = a && b; // logical & in C

endmodule
```

5. NOR gate
```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = !(a || b); // reverse the OR
endmodule
```

6. XNOR gate
```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = !(a ^ b);  // only output 1 when inputs are same
						    // opposite to XOR
						    // where output is 1 when inputs
						    // are different
endmodule
```

7. Declaring wires
```verilog
`default_nettype none
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out,
    output out_n   ); 
    wire ab_out;
    wire cd_out;
    assign ab_out = a && b; // AND gate
    assign cd_out = c && d; // AND gate
    assign out = ab_out || cd_out; // OR gate
    assign out_n = !out; // NOT gate
    
endmodule
```

8. The 7458 Chip
```verilog
module top_module ( 
    input p1a, p1b, p1c, p1d, p1e, p1f,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );
    // adding inner connectors
	wire p2ab, p2cd;
    wire p1abc, p1def;
    // connect "1" inputs to AND gates
    assign p1abc = p1a && p1b && p1c;
    assign p1def = p1d && p1e && p1f;
    // connect "2" inputs to AND gates
    assign p2ab = p2a && p2b;
    assign p2cd = p2c && p2d;
    // connect to OR gates for final outputs
    assign p1y = p1abc || p1def;
    assign p2y = p2ab || p2cd;

endmodule
```
