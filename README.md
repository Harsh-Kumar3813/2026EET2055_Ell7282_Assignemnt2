WASHING MACHINE - FIVE-STAGE PIPELINED MICROCONTROLLER
ELL7282 / ELL782 Assignment 2
Logisim Evolution 5.0.0 | 8-bit data | 16-bit instructions

Start Here
1. Extract the ZIP into an ordinary folder.
2. Open washing_pipeline.circ in Logisim Evolution 5.0.0.
3. Main is the default view: input pins are on the left, observed
   outputs on the right. The entire implementation is already wired.
   Open this fresh file in Logisim Evolution 5.0.0. RegisterBank is the
   rebuilt register-file circuit. See CIRCUIT_NAMES.txt for renamed blocks.
   There are ZERO tunnels. Named subcircuits contain editable gates/wires.
4. The wash program is embedded in PROGRAM_ROM. You do NOT need to load
   program.hex for the normal demonstration.

Using the  CLOCK AND Inputs
- Use the Poke tool (hand) to change pins. MODE is 2 bits; WATER_LEVEL,
  TEMP and DRUM_RPM are 8 bits. Multi-bit pins accept keyboard entry;
  select the pin, then enter its displayed radix value. Pin properties
  can change the radix for convenience. Right-side output buses use hex.
- Main has one GLOBAL_CLOCK. Enable ticks from the Simulate menu,
  or advance clock ticks manually. Two clock toggles form one complete
  low/high cycle; state changes on rising edges.
- Start slowly (e.g. 8 Hz clock toggles) to see stage changes. Faster
  ticking is fine for the washing demonstration. Pause ticks when changing
  sensor pins, then resume, keeping inputs stable around rising edges.
- Assert RESET=1, advance at least one rising edge, then set RESET=0.
  This resets PC, pipeline validity, general registers and peripherals.
  RAM is not reset; the wash program does not depend on RAM contents.

MODE 00 Demonstration
1. With ticks paused, reset. Set START=0, DOOR_CLOSED=1, MODE=00,
   WATER_LEVEL=99 decimal (63 hex), TEMP=0 and DRUM_RPM=0.
2. Run two complete cycles with START=0, then set START=1 and keep it high.
3. The controller accepts START, snapshots MODE, locks and fills.
   ACTUATORS=21 hex means lock + inlet. DISPLAY becomes01.
4. Leave WATER_LEVEL=99 for a while: the inlet must remain on.
   Set WATER_LEVEL=102 decimal (66 hex), demonstrating a skipped threshold.
5. Inlet closes; the timer starts. MOTOR_DIR alternates and ACTUATORS
   alternates24/2C. CPU_MOTOR_EN and ACTUAL_MOTOR should both be1 with
   the door closed and TRIP=0.
6. When PUMP_OUT=1 (ACTUATORS=30, DISPLAY=02), set WATER_LEVEL=9 (09 hex).
   Until WATER_LEVEL<=10, the controller keeps draining.
7. The pump stops; the timer is cleared/rearmed for spin. After spin:
   HALTED=1, ACTUATORS=40, DISPLAY=AA, BUZZER=1, DOOR_LOCK=0,
   CPU_MOTOR_EN=0 and ACTUAL_MOTOR=0.
8. Normal completion halts. Reset before another normal cycle.

MODE 01 Demonstration
Reset and repeat with MODE=01. Fill holds at149 decimal (95 hex) and
ends after WATER_LEVEL=153 decimal (99 hex). Drain at9 again.
Parameters: mode00 target100 / wash80 clocks / spin40 clocks;
mode01 target150 / wash120 clocks / spin60 clocks.
Modes10 and11 deliberately use the same high-mode parameters as01.
Change MODE after ACCEPT or after fill starts to demonstrate that the
accepted mode stays fixed (SNAP output), despite pin changes.
Timer durations are global-clock counts. Processor polling/setup adds latency
between reaching MATCH and software stopping a phase.

DoorSafety Demonstration
- During wash or spin, confirm CPU_MOTOR_EN=1 and ACTUAL_MOTOR=1.
- Pause clock low, open DOOR_CLOSED=0: ACTUAL_MOTOR immediately becomes0,
  before any new clock. Advance a rising edge: TRIP becomes1.
- Reclose the door: ACTUAL_MOTOR must stay0 while TRIP=1.
- Continue ticking: software detects TRIP, clears actuator requests,
  disables the timer, displaysEE and returns to its waiting loop.
- Wait until CPU_MOTOR_EN=0. Set START=0 through a rising edge, then set
  START=1 while the door is closed. Software's FA acknowledgement accepts
  this fresh safe edge, clears TRIP and snapshots MODE for a new cycle.
- Reclosing the door alone or keeping START continuously high cannot rearm.
The circuit also includes a standalone DoorSafety subcircuit for direct tests.
This is an educational digital interlock, not a certified appliance design.

Submitted Contents
washing_pipeline.circ - complete design
isa.pdf - instruction formats and encoding
architecture.pdf - pipeline and safety design
memory_map.pdf - memory and peripheral map
program.asm - commented washing program
program.hex - ROM image
listing.pdf - address/hex/assembly listing
pipeline_traces.pdf - required hazard traces for Sections 8.2-8.4
validation.pdf - six required validation groups
README.txt - these operating instructions

CIRCUIT Names
Main is the operator panel. WashController contains the processor and ROM.
Processor is the CPU; RegisterBank is the register file.
Some reports use earlier names: RUN_PANEL=Main, washing_pipeline=WashController,
CPU=Processor, REGFILE=RegisterBank, MEMORY_IO=MemoryAndIO,
SAFETY=DoorSafety, TIMER=Timer, FORWARD=Forwarding, DECODE=InstructionDecoder,
ADD8=ByteAdder. MUXn and EQn correspond to MuxnBit and EqualnBit.

REPORTS AND SCREENSHOT EVIDENCE
isa.pdf describes the instructions and their binary encodings.
architecture.pdf explains the processor, pipeline and controller design.
memory_map.pdf lists the RAM and memory-mapped I/O addresses.
listing.pdf contains the program listing and instruction encodings.
pipeline_traces.pdf shows pipeline behaviour for key test scenarios.
validation.pdf records test results and includes screenshots of
simulation states and automated test output.

program.asm contains the assembly program, and program.hex contains
the machine code for the instruction ROM. README.txt provides setup,
operation and testing instructions.

TESTING
The supplied tests cover pipeline forwarding, load-use stalls, branch
flushing, memory access, timer operation, door safety and complete
washing cycles in both modes.

The reference run passed 17 functional test groups and 1,096
register-bank checks. The connectivity audit checked 847 nets across
22 subcircuits. These checks cover the exercised scenarios rather
than every possible input sequence.

