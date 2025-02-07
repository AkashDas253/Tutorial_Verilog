# **Comprehensive Notes on Verilog Procedural Constructs**  

Verilog provides **procedural constructs** for describing digital circuits in a **behavioral** way. These constructs control **sequential execution** of statements inside procedural blocks like `always` and `initial`.  

---

## **1. Procedural Blocks**  
Verilog has two primary **procedural blocks**:  

| **Block** | **Purpose** | **Execution** |
|-----------|------------|--------------|
| `initial` | Executes **only once** at the start of the simulation | Runs **once** at time `t=0` |
| `always`  | Executes **repeatedly** as long as the simulation runs | Runs **whenever triggered** by sensitivity list |

### **Example: Initial and Always Blocks**  
```verilog
module procedural_example;
    reg clk, reset;

    // Initial block runs once
    initial begin
        clk = 0;
        reset = 1;
        #5 reset = 0; // Change reset after 5 time units
    end

    // Always block runs repeatedly
    always #10 clk = ~clk; // Clock toggles every 10 time units
endmodule
```

---

## **2. Procedural Assignments**  
Inside procedural blocks, **variables (`reg`, `integer`, etc.)** are assigned using:  

| **Type** | **Operator** | **Behavior** |
|----------|-------------|--------------|
| **Blocking** | `=`  | Executes immediately, **in order** |
| **Non-blocking** | `<=` | Executes **in parallel** (scheduled for the next time step) |

### **Example: Blocking vs. Non-blocking Assignments**  
```verilog
module assignment_example;
    reg a, b, c;

    initial begin
        // Blocking assignment (executes in order)
        a = 1;
        b = a;  // `b` gets assigned after `a`
        
        // Non-blocking assignment (executes in parallel)
        a <= 1;
        b <= a; // `b` does not get the updated `a` immediately
    end
endmodule
```
🔹 **Use `=` for combinational logic** and **`<=` for sequential logic (flip-flops).**

---

## **3. Control Flow Statements**  
Control flow statements allow **conditional execution and looping**.  

### **3.1 Conditional Statements**  
| **Statement** | **Usage** |
|--------------|----------|
| `if-else` | Executes a block **if condition is true**, else executes another block |
| `case` | Used for **multi-way branching** based on a variable's value |
| `casez` | Like `case`, but treats **`z` (high-impedance) as a wildcard** |
| `casex` | Like `case`, but treats **`x` (unknown) and `z` as wildcards** |

#### **Example: if-else Statement**  
```verilog
module if_example;
    reg a, b, c;
    
    always @(*) begin
        if (a == 1) 
            b = 1;
        else 
            b = 0;
    end
endmodule
```

#### **Example: case Statement (Multiplexer)**
```verilog
module case_example;
    reg [1:0] sel;
    reg a, b, c, d;
    wire out;

    always @(*) begin
        case (sel)
            2'b00: out = a;
            2'b01: out = b;
            2'b10: out = c;
            2'b11: out = d;
            default: out = 0;
        endcase
    end
endmodule
```
🔹 **Use `casez` or `casex` for wildcard handling.**

---

### **3.2 Loops**  
Verilog supports **looping constructs** for iterative execution.

| **Loop Type** | **Usage** |
|--------------|----------|
| `for` | Executes a loop **for a fixed number of iterations** |
| `while` | Executes **until a condition is false** |
| `repeat` | Executes a loop **a fixed number of times** |
| `forever` | Runs **indefinitely** (must use `disable` or `break`) |

#### **Example: for Loop**  
```verilog
module for_loop_example;
    integer i;
    reg [7:0] memory [0:15];

    initial begin
        for (i = 0; i < 16; i = i + 1)
            memory[i] = i; // Initialize memory
    end
endmodule
```

#### **Example: while Loop**  
```verilog
module while_example;
    integer i = 0;
    
    initial begin
        while (i < 10) begin
            $display("Iteration: %d", i);
            i = i + 1;
        end
    end
endmodule
```

#### **Example: repeat Loop**  
```verilog
module repeat_example;
    integer i = 0;

    initial begin
        repeat (5) begin
            $display("This prints 5 times");
            i = i + 1;
        end
    end
endmodule
```

#### **Example: forever Loop (Clock Generation)**
```verilog
module forever_example;
    reg clk;

    initial begin
        forever #5 clk = ~clk; // Toggle clock every 5 time units
    end
endmodule
```

---

## **4. Tasks and Functions**  
Verilog supports **modular programming** using **tasks** and **functions**.

| **Construct** | **Purpose** | **Supports `#delay`?** | **Returns a Value?** |
|--------------|------------|----------------|----------------|
| `task` | Groups multiple procedural statements | ✅ Yes | ❌ No |
| `function` | Computes and returns a value | ❌ No | ✅ Yes |

#### **Example: Task (Without Return Value)**
```verilog
module task_example;
    task display_message;
        input integer x;
        begin
            $display("Value: %d", x);
        end
    endtask
    
    initial begin
        display_message(10); // Calls the task
    end
endmodule
```

#### **Example: Function (With Return Value)**
```verilog
module function_example;
    function integer add;
        input integer a, b;
        begin
            add = a + b; // Returns sum
        end
    endfunction

    initial begin
        $display("Sum = %d", add(3, 5));
    end
endmodule
```

---

## **5. Fork-Join for Parallel Execution**  
Verilog allows **parallel execution** using `fork-join`.

| **Type** | **Behavior** |
|----------|-------------|
| `fork-join` | Executes all statements **in parallel**, waits for all to finish |
| `fork-join_any` | Executes all statements **in parallel**, waits for **any one** to finish |
| `fork-join_none` | Executes all statements **in parallel**, does not wait |

### **Example: Parallel Execution Using Fork-Join**
```verilog
module fork_example;
    initial begin
        fork
            #5 $display("Task 1");
            #10 $display("Task 2");
            #15 $display("Task 3");
        join
    end
endmodule
```
🔹 **Use `join_any` or `join_none` for better control over execution.**

---

## **Conclusion**  
Verilog **procedural constructs** help model **sequential** and **conditional** behaviors.  
- Use `initial` for **one-time execution**.  
- Use `always` for **repeated execution** (clocked or combinational logic).  
- Use `blocking (=)` for **combinational logic** and `non-blocking (<=)` for **sequential logic**.  
- Use **control flow (`if`, `case`, loops)** for **behavioral modeling**.  
- Use **tasks and functions** for **modular and reusable code**.  
- Use **fork-join** for **parallel execution**.  

🚀 Mastering these constructs is essential for designing **efficient and structured** Verilog code!