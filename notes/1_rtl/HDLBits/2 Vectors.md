1. Vectors introduction

`Vectors are used to group related signals using one name to make it more convenient to manipulate. For example, wire [7:0] w; declares an 8-bit vector named w that is functionally equivalent to having 8 separate wires.

Notice that the _declaration_ of a vector places the dimensions _before_ the name of the vector, which is unusual compared to C syntax. However, the _part select_ has the dimensions _after_ the vector name as you would expect.

```verilog
wire **[99:0]** my_vector; // Declare a 100-element vector 
assign out = my_vector[10]; // Part-select one bit out of the vector
```

![[Pasted image 20261005074513.png]]

```verilog
module top_module ( 
    input wire [2:0] vec,
    output wire [2:0] outv,
    output wire o2,
    output wire o1,
    output wire o0  ); // Module body starts after module declaration
	
    // vector to vector output
    assign outv = vec;
    
    // vector to slices (here are bits)
    assign o2 = vec[2];
    assign o1 = vec[1];
    assign o0 = vec[0];
    // equivalent: ssign {o2, o1, o0} = vec;
endmodule
```


### Declaring Vectors

Vectors must be declared:

type [upper:lower] vector_name;

type specifies the datatype of the vector. This is usually wire or reg. If you are declaring a input or output port, the type can additionally include the port type (e.g., input or output) as well. Some examples:

```verilog
wire [7:0] w;         // 8-bit wire
reg  [4:1] x;         // 4-bit reg
output reg [0:0] y;   // 1-bit reg that is also an output port (this is still a vector)
input wire [3:-2] z;  // 6-bit wire input (negative ranges are allowed)
output [3:0] a;       // 4-bit output wire. Type is 'wire' unless specified otherwise.
wire [0:7] b;         // 8-bit wire where b[0] is the most-significant bit.

```

#### "Endianness" (big / little) - the "direction"

**least significant bit** has a lower index (little-endian, e.g., [3:0]) 
**least significant bit** has a higher index (big-endian, e.g., [0:3]).

In Verilog, once a vector is declared with a particular endianness, it must always be used the same way. e.g., writing `vec[0:3]` when `vec` is declared `wire [3:0] vec;` is illegal. Being consistent with endianness is good practice, as weird bugs occur if vectors of different endianness are assigned or used together.

2. Build a combinational circuit that splits an input half-word (16 bits, [15:0] ) into lower [7:0] and upper [15:8] bytes.

```verilog
`default_nettype none     // Disable implicit nets. Reduces some types of bugs.
module top_module( 
    input wire [15:0] in,
    output wire [7:0] out_hi,
    output wire [7:0] out_lo );
    assign out_hi = in[15:8];
    assign out_lo = in[7:0];

endmodule
```

#### Part-select of vectors

A 32-bit vector can be viewed as containing 4 bytes (bits [31:24], [23:16], etc.). 

AaaaaaaaBbbbbbbbCcccccccDddddddd => DdddddddCcccccccBbbbbbbbAaaaaaaa

This operation is often used when the [endianness](https://en.wikipedia.org/wiki/Endianness) of a piece of data needs to be swapped, for example between little-endian x86 systems and the big-endian formats used in many Internet protocols.

3. Build a circuit that will reverse the _byte_ ordering of the 4-byte word.

```verilog
module top_module( 
    input [31:0] in,
    output [31:0] out );//

    // assign out[31:24] = ...;
    assign out[31:24] = in[7:0];
    assign out[23:16] = in[15:8];
    assign out[15:8] = in[23:16];
    assign out[7:0] = in[31:24];
    
endmodule

```

4. Build a circuit that has two 3-bit inputs that computes the bitwise-OR of the two vectors, the logical-OR of the two vectors, and the inverse (NOT) of both vectors. Place the inverse of `b` in the upper half of `out_not` (i.e., bits [5:3]), and the inverse of `a` in the lower half.

![[Pasted image 20261008003313.png]]

```verilog
module top_module( 
    input [2:0] a,
    input [2:0] b,
    output [2:0] out_or_bitwise,
    output out_or_logical,
    output [5:0] out_not
);

    assign out_or_bitwise[2:0] = a|b;
    assign out_or_logical = a||b;
    assign out_not[5:0] = {~b,~a};
    //equivalent:
    // assign out_not[5:3] = ~b;
    // assign out_not[2:0] = ~a;
endmodule

```

Look at the simulation waveforms at how the bitwise-OR and logical-OR differ.

![[Pasted image 20261008003450.png]]

5. Build a combinational circuit with four inputs, in[3:0].

There are 3 outputs:

- out_and: output of a 4-input AND gate.
- out_or: output of a 4-input OR gate.
- out_xor: output of a 4-input XOR gate.

```verilog
module top_module( 
    input [3:0] in,
    output out_and,
    output out_or,
    output out_xor
);
	//using short-hand notations:
    assign out_and = &in;
    assign out_or = |in;
    assign out_xor = ^in;
    
 	// fully-expanded equivalents:
	//    assign out_and = in[0]&in[1]&in[2]&in[3];
	//    assign out_or = in[0]|in[1]|in[2]|in[3];
	//    assign out_xor = in[0]^in[1]^in[2]^in[3];
    
endmodule
```

#### The concatenation operator

 The concatenation operator {a,b,c} is used to create larger vectors by concatenating smaller portions of a vector together.

{3'b111, 3'b000} => 6'b111000
{1'b1, 1'b0, 3'b101} => 5'b10101
{4'ha, 4'd10} => 8'b10101010     // 4'ha and 4'd10 are both 4'b1010 in binary

6. Given several input vectors, concatenate them together then split them up into several output vectors. There are six 5-bit input vectors: a, b, c, d, e, and f, for a total of 30 bits of input. There are four 8-bit output vectors: w, x, y, and z, for 32 bits of output. The output should be a concatenation of the input vectors followed by two 1 bits:

![[Pasted image 20261008003806.png]]

```verilog
module top_module (
    input [4:0] a, b, c, d, e, f,
    output [7:0] w, x, y, z );//

    // assign { ... } = { ... };
    assign {w,x,y,z} = {a,b,c,d,e,f,1'b1,1'b1};
    // syntax for raw bits is 
    // <width>'<base><value>
    // e.g. 4'b0001 = 0001 (if only least significant bit can shorten to 4'b1)
    // e.g. 4'd10 = 1010 (d is decimal, also accept "o" = octo and "h" = hex)
    // below would also work:
    // assign {w[7:0],x[7:0],y[7:0],z[7:0]} = {a[4:0],b[4:0],c[4:0],d[4:0],e[4:0],f[4:0],1'b1,1'b1};

endmodule
```

7. Given an 8-bit input vector [7:0], reverse its bit ordering.

```verilog
module top_module( 
    input [7:0] in,
    output [7:0] out
);
    assign out = {in[0],in[1],in[2],in[3],in[4],in[5],in[6],in[7]};
    
endmodule
```

8. Build a circuit that sign-extends an 8-bit number to 32 bits. This requires a concatenation of 24 copies of the sign bit (i.e., replicate bit[7] 24 times) followed by the 8-bit number itself.

```verilog
module top_module (
    input [7:0] in,
    output [31:0] out );//

    // assign out = { replicate-sign-bit , the-input };
    assign out = {{24{in[7]}}, in};
    // remember the part to be replicated
    // should be enclosed by {} e.g. {in[7]}
    // so it is { 24{in[7]} }
endmodule
```

9. Given five 1-bit signals (a, b, c, d, and e), compute all 25 pairwise one-bit comparisons in the 25-bit output vector. The output should be 1 if the two bits being compared are equal.


	out[24] = ~a ^ a;   // a == a, so out[24] is always 1.
	out[23] = ~a ^ b;
	out[22] = ~a ^ c;
	...
	out[ 1] = ~e ^ d;
	out[ 0] = ~e ^ e;
	
![[Pasted image 20261008010127.png]]

```verilog
module top_module (
    input a, b, c, d, e,
    output [24:0] out );//

    // The output is XNOR of two vectors created by 
    // concatenating and replicating the five inputs.
    // assign out = ~{ ... } ^ { ... };
    assign out = ~{{5{a}},{5{b}},{5{c}},{5{d}},{5{e}}} ^ {{5{a,b,c,d,e}}};

endmodule
```