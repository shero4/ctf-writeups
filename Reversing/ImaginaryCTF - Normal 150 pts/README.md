# Normal - 150pts
## Reversing, Verilog, Python
![Screenshot 2021-07-28 at 9.01.42 AM.png](https://www.dropbox.com/s/ubb6c4vpbbl4umt/Screenshot%202021-07-28%20at%209.01.42%20AM.png?dl=0&raw=1)

## normal.v
```verilog
module normal(out, in);
    output [255:0] out;
    input [255:0] in;
    wire [255:0] w1, w2, w3, w4, w5, w6, w7, w8; 
 
    wire [255:0] c1, c2;
    assign c1 = 256'h44940e8301e14fb33ba0da63cd5d2739ad079d571d9f5b987a1c3db2b60c92a3;
    assign c2 = 256'hd208851a855f817d9b3744bd03fdacae61a70c9b953fca57f78e9d2379814c21;
    
    nor n1 [255:0] (w1, in, c1);
    nor n2 [255:0] (w2, in, w1);
    nor n3 [255:0] (w3, c1, w1);
    nor n4 [255:0] (w4, w2, w3);
    nor n5 [255:0] (w5, w4, w4);
    nor n6 [255:0] (w6, w5, c2);
    nor n7 [255:0] (w7, w5, w6);
    nor n8 [255:0] (w8, c2, w6);
    nor n9 [255:0] (out, w7, w8);
endmodule

module main;
    wire [255:0] flag = 256'h696374667b00000000000000000000000000000000000000000000000000007d;
    wire [255:0] wrong;

    normal flagchecker(wrong, flag);

    initial begin
        #10;
        if (wrong) begin
            $display("Incorrect flag...");
            $finish;
        end
        $display("Correct!");
    end
endmodule
```
We are given a verilog file with two modules, main and normal. The main module creates an instance of the normal module, if normal flagchecker returns 1, flag is correct else incorrect is displayed

I started off by converting all the nor gates into one single gate by hand(NOR is represented by a . and NOT is represented by ~ for simplicity)
```
 nor n1 [255:0] (w1, a, b); // a.b
 nor n2 [255:0] (w2, a, w1); // a.(a.b) 
 nor n3 [255:0] (w3, b, w1); // b.(a.b)
 nor n4 [255:0] (w4, w2, w3); // (a.(a.b)).(b.(a.b))
 nor n5 [255:0] (w5, w4, w4); // ~((a.(a.b)).(b.(a.b)))
 nor n6 [255:0] (w6, w5, c); // (~((a.(a.b)).(b.(a.b)))).c
 nor n7 [255:0] (w7, w5, w6); // (~((a.(a.b)).(b.(a.b)))).((~((a.(a.b)).(b.(a.b)))).c)
 nor n8 [255:0] (w8, c, w6); // c.((~((a.(a.b)).(b.(a.b)))).c)
 nor n9 [255:0] (out, w7, w8); // ((~((a.(a.b)).(b.(a.b)))).((~((a.(a.b)).(b.(a.b)))).c)).(c.((~((a.(a.b)).(b.(a.b)))).c))
```
 Then I put the final expression into wolfram alpha https://www.wolframalpha.com/widgets/view.jsp?id=a52797be9f91295a27b14cb751198ae3 and recieved a simplified version of it.
 
 I honestly didnt know how bitwise not could be done in python so I replaced all the not expressions with xor 1 which does the same thing. a= input, b=c1, c=c2
 
 ```
 (a & b & (c^1)) | (a & (b^1) & c) | ((a^1) & b & c) | ((a^1) & (b^1) & (c^1))
 ```
 Now I quickly wrote a python script to bruteforce this gate against every printable charecter and got the flag
 
 ```python
 import string

def compute(chr, n):
    for j in range(0, 8): # each bit in possible letter
        a = (ord(chr) >> 7-j) & 1
        b = (0x44940e8301e14fb33ba0da63cd5d2739ad079d571d9f5b987a1c3db2b60c92a3 >> (
            255 - (8*n) - j)) & 1
        c = (0xd208851a855f817d9b3744bd03fdacae61a70c9b953fca57f78e9d2379814c21 >> (
            255 - (8*n) - j)) & 1
        final = (a & b & (c^1)) | (a & (b^1) & c) | ((a^1) & b & c) | ((a^1) & (b^1) & (c^1))
        if final != 0:
            return False
    return True

def main():
    final = ""
    n = 0
    for _ in range(0,32): # each byte of flag
        for chr in string.printable: # each possible letter
            if compute(chr, n):
                final += chr
                break
        n += 1
    print(final)

main()
 ```
 
 > flag: ictf{A11_ha!1_th3_n3w_n0rm_n0r!}
