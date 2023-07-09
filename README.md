vpp verilog per processing by perl, 一个perl实现的verilog预处理脚本
维护和修改了一个便捷的Verilog SystermVerilog 预处理脚本。 方便之处在于可以用perl强大的字符处理功能批量实现Verilog代码的生成。 我个人使用下来几点好处
0. vpp.pl 脚本完全兼容原生verilog/systemVerilog 代码， 就是说现有的verilog代码处理后不变化。 但如果在现有代码中出现一下四种种情况则vpp.pl脚本会被唤醒并作相应处理；


  a. //@ ...        => //@ 之后的任何文本直到本行结束（/n） （注意本行后的回车会被消除）


  b. /*@ ... @*/    => /*@ 和 @*/ 之间的任何文本，可以跨行或者同一行之内。 （注意中间不要被其他*/拦截）


  c. $... or ${...} => (“...”中没有空格) 变量求值， 当前perl语境中的这个变量的值输出出来


  d. $${...}        => 等价于/*@...@*/在同一行并且看作一个是eval() 一个立即数（或字符串）比如： $${$t*2+5}


##本方法在过去的十年中在多个芯片项目中使用并参与了十多次流片量产，个人认为极大的加速了数字电路设计和增强了设计的可靠性，可拓展性和调试便修改便捷性， 具体而言优势如下：

//@for 循环类似generate的功能但是要强大很多， 不用局限于定义好的数据大小。可以生成变量变化的名。 避免了铭记复杂的Verilog语法和编译其注意事项。
方便的常数计算，不用复杂的Verilog语法而是用perl 的$ver直接生成立即数（需要注释可以添加以防混淆）
增加了常用类似实例化和接口连线的功能
总体而言，使能了采用数据库第三范式的思路来编写Verilog代码，让调试修改，关键逻辑尽量只出现在一处然后通过perl变量或函数来复用，大大降低了Verilog代码的不一致产生的bug。在很多多维度遍历的使用中，Perl中的调试好的多层for循环在目标Verilog生成后就可以确认是否正确，而不像Verilog genrate语句如果是runtime error则需要在调试仿真过程中才有可能发现问题。而且这列问题往往会被其他仿真问题掩盖而浪费大量debug时间。

##典型案例：
你在写一个Verilog 或者SystemVerilog 代码加perl的混合代码 test.pv 内容如下：
/*@
my $t=3; $c=4;
my $width = 16; 
my $nIn   = 3;
@*/
//////////////////////
// $t

module abc (
//@ for my $i (1..$nIn) {
    input [$width-1:0] in$i,
    //@}  
    output [$${$width*$nIn-1} :0] out
)
//@ my $n =  "smth";

wire  [$width*$nIn-1 :0] $n;
// assign $n = {/*@for my $i (reverse 1..$nIn) {@*/ in$i , /*@}@*/};
// above have problem since there is a "," at end of code, try this:
assign $n = {/*@for my $i (reverse 1..$nIn) { my  $cm= $i==1 ? "" :","; @*/ in$i $cm /*@}@*/};

assign out = $n; 

// some other demo here commented to avoid verilog syntax:  $n $${$t};
// adf asdf   $_
// $${$t *2+5+7+$t}
// $${$t+$c . "dd" .log($c)/log(2)}
 
endmodule
用 vpp.pl -perl test.pv 处理后输出如下：
//////////////////////
// 3

module abc (
    input [16-1:0] in1,
    input [16-1:0] in2,
    input [16-1:0] in3,
    output [47 :0] out
)

wire  [16*3-1 :0] smth;
// assign smth = { in3 ,  in2 ,  in1 , };
// above have problem since there is a "," at end of code, try this:
assign smth = { in3 ,  in2 ,  in1  };

assign out = smth;

// some other demo here commented to avoid verilog syntax:  smth 3;
// adf asdf   
// 21
// 7dd2

endmodule
-- by liberalmeng@hotmail.comn 2023/04/29

