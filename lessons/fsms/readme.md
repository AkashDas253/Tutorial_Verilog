# **Finite State Machines (FSMs) in Verilog**

## **1. Introduction to FSMs**
A **Finite State Machine (FSM)** is a **sequential circuit** used to model systems that transition between a **finite number of states** based on **input conditions** and a **clock signal**.

FSMs are widely used in **control circuits**, **protocol design**, **traffic lights**, **CPU instruction execution**, and more.

---

## **2. Types of FSMs**
There are **two main types** of FSMs:

| **FSM Type** | **Definition** | **Output Depends On** | **Example** |
|-------------|---------------|----------------|----------------|
| **Moore FSM** | Output is based **only on the current state** | Current state | Traffic lights, elevator control |
| **Mealy FSM** | Output is based on **both the current state and inputs** | Current state + inputs | Sequence detectors, vending machines |

---

## **3. FSM Design Process**
To design an FSM in Verilog, follow these steps:

1. **Identify States:** Define all states the system will have.
2. **Determine Inputs and Outputs:** Define conditions that cause state transitions.
3. **State Transition Diagram:** Draw a diagram showing states and transitions.
4. **State Encoding:** Assign binary values to each state.
5. **Implement in Verilog:** Use **always** blocks, `case` statements, and registers to code the FSM.
6. **Test with a Testbench:** Simulate to verify behavior.

---

## **4. State Representation and Encoding**
Each state in an FSM is represented using **binary encoding**:

| **State Name** | **Binary Encoding (2-bit example)** |
|--------------|--------------------------------|
| **IDLE** | `00` |
| **START** | `01` |
| **PROCESS** | `10` |
| **DONE** | `11` |

🔹 **Encoding Methods:**
- **Binary Encoding:** Uses minimal bits (`n` states → `log₂(n)` bits).
- **One-Hot Encoding:** Each state has a **separate bit** (`n` states → `n` bits).
- **Gray Code Encoding:** States differ by only **one bit** to reduce glitches.

---

## **5. Moore FSM Example**
A Moore FSM outputs **depend only on the current state**.

### **Example: 2-bit Up Counter FSM**
```verilog
module moore_fsm(
    input clk, reset,
    output reg [1:0] state
);
    // Define state encoding
    parameter S0 = 2'b00, S1 = 2'b01, S2 = 2'b10, S3 = 2'b11;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= S0;  // Reset to initial state
        else begin
            case (state)
                S0: state <= S1;
                S1: state <= S2;
                S2: state <= S3;
                S3: state <= S0;
                default: state <= S0;
            endcase
        end
    end
endmodule
```
✔ **Characteristics of Moore FSM:**
- Output depends only on the **current state**.
- Easier to **design and debug**.
- Can be **slower**, as state transitions affect output changes.

---

## **6. Mealy FSM Example**
A Mealy FSM's **output depends on both current state and input**.

### **Example: Sequence Detector (Detects "101")**
```verilog
module mealy_fsm (
    input clk, reset, in,
    output reg out
);
    // Define states
    parameter S0 = 2'b00, S1 = 2'b01, S2 = 2'b10;
    reg [1:0] state;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= S0;
        else begin
            case (state)
                S0: state <= (in) ? S1 : S0;
                S1: state <= (in) ? S1 : S2;
                S2: state <= (in) ? S0 : S0;
                default: state <= S0;
            endcase
        end
    end

    // Output logic (Mealy: depends on state + input)
    always @(state, in) begin
        case (state)
            S0: out = 0;
            S1: out = 0;
            S2: out = (in) ? 1 : 0;  // Detects "101" sequence
            default: out = 0;
        endcase
    end
endmodule
```
✔ **Characteristics of Mealy FSM:**
- Output changes **immediately** with input.
- **More responsive** than Moore FSM.
- **Can have glitches** if inputs change near clock edges.

---

## **7. FSM State Transition Diagram**
FSM behavior is **best represented using a state transition diagram**.

### **Example: Moore FSM for 2-bit Counter**
```
   S0 (00) --> S1 (01) --> S2 (10) --> S3 (11) --> S0 (00)
```
Each arrow represents **state transitions** based on **clock cycles**.

### **Example: Mealy FSM for Sequence Detector**
```
          +-----> S0 (Waiting) ----+
          | 0                     | 1
          v                        v
          S1 (Got '1') --0--> S2 (Got '10')
                          | 1
                          v
                        Output '1'
```
🔹 **Diagrams help visualize FSM transitions** easily.

---

## **8. FSM Implementation Techniques**
FSMs can be implemented using:
1. **Case Statement** (Recommended for small FSMs)
2. **`if-else` Ladder** (Useful for simple FSMs)
3. **State Registers & Next-State Logic** (Best for complex FSMs)

### **Example: FSM using Next-State Logic**
```verilog
module fsm_example(
    input clk, reset, in,
    output reg out
);
    typedef enum logic [1:0] {S0, S1, S2} state_t;
    state_t state, next_state;

    // Next-state logic
    always @(*) begin
        case (state)
            S0: next_state = (in) ? S1 : S0;
            S1: next_state = (in) ? S1 : S2;
            S2: next_state = (in) ? S0 : S0;
            default: next_state = S0;
        endcase
    end

    // State transition
    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= S0;
        else
            state <= next_state;
    end
endmodule
```
✔ **Advantages of Next-State Logic:**
- **Separates state transition and output logic**, making it **modular**.
- Easily scalable for **complex FSMs**.

---

## **9. FSM Testbench**
To verify FSM behavior, use a **testbench**.

```verilog
module tb_fsm;
    reg clk, reset, in;
    wire out;

    mealy_fsm uut (
        .clk(clk), .reset(reset), .in(in), .out(out)
    );

    initial begin
        clk = 0; reset = 1; in = 0;
        #10 reset = 0; // Release reset
        #10 in = 1; #10 in = 0; #10 in = 1; // Input pattern "101"
        #10 $finish;
    end

    always #5 clk = ~clk; // Clock toggle every 5 time units
endmodule
```
✔ **Testbench ensures FSM transitions are correct** before synthesis.

---

## **10. Conclusion**
- FSMs are essential for **sequential circuit design**.
- **Moore FSMs**: Simpler, more stable, but **slower**.
- **Mealy FSMs**: More **responsive**, but **prone to glitches**.
- FSMs can be implemented using **case statements, next-state logic, or `if-else`**.
- **Testbenches verify correctness** before FPGA/ASIC synthesis.
