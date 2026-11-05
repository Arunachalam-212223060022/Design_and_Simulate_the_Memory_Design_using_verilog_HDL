# Design and Simulate the Memory Design Using Verilog HDL

## EXP NO: 5 – Design and Simulate the Memory Design Using Verilog HDL

### AIM

To design and simulate RAM, ROM, and FIFO using Verilog HDL, and verify their functionality through a testbench in the Vivado 2023.1 environment.

### APPARATUS REQUIRED

- Vivado 2023.1

### PROCEDURE

1. **Launch Vivado 2023.1:** Open Vivado and create a new project.
2. **Design the Verilog Code:** Write the Verilog code for RAM, ROM, and FIFO.
3. **Create the Testbench:** Write a testbench to simulate the memory behavior. The testbench should apply various inputs and monitor the corresponding outputs.
4. **Create the Verilog Files:** Create both the design module and the testbench in the Vivado project.
5. **Run Simulation:** Run the behavioral simulation to verify the output.
6. **Observe the Waveforms:** Analyze the output waveforms in the simulation window and verify the correct read and write operations.
7. **Save and Document Results:** Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

---

## 1. RAM – 4 KB Memory

### Verilog Code

```verilog
module memory_4KB (
    input        clk,
    input        we,          // Write Enable
    input        re,          // Read Enable
    input  [11:0] address,    // 12-bit address: 0 to 4095
    input  [7:0]  data_in,    // 8-bit input data
    output reg [7:0] data_out // 8-bit output data
);

    // 4 KB memory = 4096 locations × 8 bits
    reg [7:0] memory [0:4095];

    // Write operation
    always @(posedge clk) begin
        if (we)
            memory[address] <= data_in;
    end

    always @(*) begin
        if (re)
            data_out = memory[address];
        else
            data_out = 8'bz;
    end

endmodule
```

### Testbench – RAM

```verilog
module tb_memory_4KB;

    reg        clk;
    reg        we;
    reg        re;
    reg [11:0] address;
    reg [7:0]  data_in;

    wire [7:0] data_out;

    // Instantiate DUT
    memory_4KB DUT (
        .clk      (clk),
        .we       (we),
        .re       (re),
        .address  (address),
        .data_in  (data_in),
        .data_out (data_out)
    );

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    // Test procedure
    initial begin

        // Initialize signals
        we      = 0;
        re      = 0;
        address = 12'd0;
        data_in = 8'd0;

        // -------------------------------
        // WRITE OPERATION
        // -------------------------------

        // Write 55H into address 100
        #10;
        we      = 1;
        address = 12'd100;
        data_in = 8'h55;

        #10;
        we = 0;

        // Write AAH into address 200
        #10;
        we      = 1;
        address = 12'd200;
        data_in = 8'hAA;

        #10;
        we = 0;

        // Write FFH into last address
        #10;
        we      = 1;
        address = 12'd4095;
        data_in = 8'hFF;

        #10;
        we = 0;

        // -------------------------------
        // READ OPERATION
        // -------------------------------

        // Read address 100
        #10;
        re      = 1;
        address = 12'd100;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Disable read
        #10;
        re = 0;

        #10;
        $finish;

    end

endmodule
```

### Output Waveform – RAM

<img width="1920" height="1080" alt="RAM Output Waveform" src="https://github.com/user-attachments/assets/245b6f17-630f-4f45-9952-dc652c683d9b" />

---

## 2. ROM – 4 KB Memory

### Verilog Code

```verilog
`timescale 1ns/1ps

module rom_4KB (
    input        [11:0] address,
    input        re,
    output reg   [7:0] data_out
);

    // 4 KB ROM = 4096 locations × 8 bits
    reg [7:0] memory [0:4095];

    // Initialize ROM contents
    initial begin
        memory[0] = 8'h10;
        memory[1] = 8'h20;
        memory[2] = 8'h30;
        memory[3] = 8'h40;
        memory[4] = 8'h50;
        memory[5] = 8'h60;
        memory[6] = 8'h70;
        memory[7] = 8'h80;

        memory[100]  = 8'h55;
        memory[200]  = 8'hAA;
        memory[4095] = 8'hFF;
    end

    // Read operation
    always @(*) begin
        if (re)
            data_out = memory[address];
        else
            data_out = 8'bz;
    end

endmodule
```

### Testbench – ROM

```verilog
`timescale 1ns/1ps

module tb_rom_4KB;

    reg        re;
    reg [11:0] address;

    wire [7:0] data_out;

    // Instantiate DUT
    rom_4KB DUT (
        .address  (address),
        .re       (re),
        .data_out (data_out)
    );

    initial begin

        // Initialize signals
        re      = 0;
        address = 12'd0;

        // -------------------------------
        // READ OPERATION
        // -------------------------------

        // Read address 0
        #10;
        re      = 1;
        address = 12'd0;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Read address 1
        #10;
        address = 12'd1;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Read address 2
        #10;
        address = 12'd2;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Read address 100
        #10;
        address = 12'd100;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Read address 200
        #10;
        address = 12'd200;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Read last address
        #10;
        address = 12'd4095;

        #5;
        $display("Address = %d, Data = %h", address, data_out);

        // Disable read
        #10;
        re = 0;

        #10;
        $finish;

    end

endmodule
```

### Output Waveform – ROM

<img width="1920" height="1080" alt="ROM Output Waveform" src="https://github.com/user-attachments/assets/83b4d92a-7edf-491d-ada7-acb35dd1043f" />

---

## 3. FIFO – First In First Out

### Verilog Code

```verilog
`timescale 1ns/1ps

module fifo (
    input        clk,
    input        reset,
    input        wr_en,
    input        rd_en,
    input  [7:0] data_in,
    output reg [7:0] data_out,
    output wire full,
    output wire empty
);

    // 8 locations × 8 bits
    reg [7:0] memory [0:7];

    // Read and write pointers
    reg [3:0] write_ptr;
    reg [3:0] read_ptr;

    // Number of stored elements
    reg [3:0] count;

    // FIFO status
    assign full  = (count == 4'd8);
    assign empty = (count == 4'd0);

    always @(posedge clk) begin

        if (reset) begin

            write_ptr <= 4'd0;
            read_ptr  <= 4'd0;
            count     <= 4'd0;
            data_out  <= 8'd0;

        end
        else begin

            // Write operation
            if (wr_en && !full) begin
                memory[write_ptr] <= data_in;
                write_ptr <= write_ptr + 1'b1;
            end

            // Read operation
            if (rd_en && !empty) begin
                data_out <= memory[read_ptr];
                read_ptr <= read_ptr + 1'b1;
            end

            // Update count
            case ({wr_en && !full, rd_en && !empty})

                2'b10: count <= count + 1'b1;

                2'b01: count <= count - 1'b1;

                default: count <= count;

            endcase

        end

    end

endmodule
```

### Testbench – FIFO

```verilog
`timescale 1ns/1ps

module tb_fifo;

    reg        clk;
    reg        reset;
    reg        wr_en;
    reg        rd_en;
    reg  [7:0] data_in;

    wire [7:0] data_out;
    wire       full;
    wire       empty;

    // Instantiate DUT
    fifo DUT (
        .clk      (clk),
        .reset    (reset),
        .wr_en    (wr_en),
        .rd_en    (rd_en),
        .data_in  (data_in),
        .data_out (data_out),
        .full     (full),
        .empty    (empty)
    );

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin

        // Initialize signals
        reset   = 1;
        wr_en   = 0;
        rd_en   = 0;
        data_in = 8'h00;

        // Reset FIFO
        #10;
        reset = 0;

        // --------------------------------
        // WRITE DATA
        // --------------------------------

        // Write 11H
        #10;
        wr_en   = 1;
        data_in = 8'h11;

        #10;
        // Write 22H
        data_in = 8'h22;

        #10;
        // Write 33H
        data_in = 8'h33;

        #10;
        // Write 44H
        data_in = 8'h44;

        #10;
        wr_en = 0;

        // --------------------------------
        // READ DATA
        // --------------------------------

        #10;
        rd_en = 1;

        #10;
        $display("Read Data = %h", data_out);

        #10;
        $display("Read Data = %h", data_out);

        #10;
        $display("Read Data = %h", data_out);

        #10;
        $display("Read Data = %h", data_out);

        #10;
        rd_en = 0;

        // --------------------------------
        // DISPLAY FIFO STATUS
        // --------------------------------

        #10;
        $display("FIFO Empty = %b", empty);
        $display("FIFO Full  = %b", full);

        #10;
        $finish;

    end

endmodule
```

### Output Waveform – FIFO

<img width="1920" height="1080" alt="FIFO Output Waveform" src="https://github.com/user-attachments/assets/5d67bc82-37b1-41a1-8a94-288dfe74f420" />

---

## CONCLUSION

The RAM, ROM, and FIFO memories with read and write operations were designed and successfully simulated using Verilog HDL. The testbench verified the read and write functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
