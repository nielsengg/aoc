# Testando seu RISC-V monociclo na placa

O objetivo desta prática é testar diretamente no kit de FPGA [o processador que você desenvolveu na simulação](https://github.com/DC-UFSCar/riscv-mono) desta semana. Para isso, foi implementado externamente (no SoC[^1]) um esquema de E/S mapeados em memória. Para possibilitar o monociclo, separamos as memórias de instruções e de dados:

```verilog
  // microprocessor
  riscvmono cpu(clk, reset, pc, instr, addr, writedata, memwrite, readdata);

  // instructions memory 
  rom instr_mem(pc, instr);

  // data memory 
  ram data_mem(clk, memwrite & isRAM, addr, writedata, readdata);
```

A memória de instruções é uma ROM que recebe o PC e retornar a instrução correspondente ao endereço. Já a de dados é uma RAM que armazena os dados do programa, que podem ser lidos (`lw`) ou escritos (`sw`). Cada uma delas possui apenas 256 posições, ou seja, podem ser endereçadas a partir de um único byte. 

O esquema de E/S mapeado em memória é apresentado a seguir:

```verilog
  // memory-mapped i/o
  wire isIO  = addr[8]; // 0x0000_0100
  wire isRAM = !isIO;
  localparam IO_LEDS_bit = 2; // 0x0000_0104
  localparam IO_HEX_bit  = 3; // 0x0000_0108
  reg [23:0] hex_digits; // memory-mapped I/O register for HEX
  dec9segs hex0(hex_digits[ 3: 0], HEX0);
  dec9segs hex1(hex_digits[ 7: 4], HEX1);
  dec9segs hex2(hex_digits[11: 8], HEX2);
  dec9segs hex3(hex_digits[15:12], HEX3);
  dec9segs hex4(hex_digits[19:16], HEX4);
  dec9segs hex5(hex_digits[23:20], HEX5);
  always @(posedge clk)
    if (memwrite & isIO) begin // I/O write 
      if (addr[IO_LEDS_bit])
        LEDR <= writedata;
      if (addr[IO_HEX_bit])
        hex_digits <= writedata;
  end
``` 

[^1]: SoC significa *System on Chip*. No contexto deste projeto ele se refere ao arquivo *top level* para síntese, que contém a CPU, memórias de instruções e de dados, além dos periféricos, neste caso os LEDs e *displays* de sete segmentos. 