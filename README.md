🚀 RISC-V Bare-metal Dual-Mode Toolchain (riscv-tool)
riscv-tool is an ultra-lightweight bare-metal execution and development tool for the RISC-V (RV64GC) target. Within a single tool, it fully supports an ultra-lightweight bare-metal C environment and Standard C++ / Newlib stdio.h supported C++ environments through conditional builds (feature toggling) and source file extension detection.
✨ Key Features⚡
Dual-Mode Architecture:
Lite Mode: Bare-metal C development with pure assembly and a minimal footprint without the standard library (-nostdlib).
Full Mode: Support for the Newlib C library (stdio.h) and C++ runtime environment.
🔄 Automatic Compiler Selection:
.c source file $\rightarrow$ riscv-none-elf-gcc.cpp /
.cc source file $\rightarrow$ riscv-none-elf-g++
🛠️ Embedded C++ Support: Global objects Constructor/Destructor Support (Accepts .init_array linker section and __libc_init_array runtime)
Full support for virtual functions and polymorphism (vtable)
Support for sbrk stub-based dynamic memory allocation (new / delete) 🖥️
Seamless QEMU Emulation: Provides system call stubs optimized for the QEMU virt platform (UART0 IO register: 0x10000000)
Clean session termination via SiFive Test Finisher 🏗️
System Architecture
Plaintext [ User Source ] (.c / .cpp)

│

▼

[ Compiler Engine ]

├── Extension Detection (.cpp -> g++, .c -> gcc)

├── Feature Check (Lite vs Full)

└── Dynamic Code Injection (c_runtime_header + link.ld)

│

▼

[ RISC-V Toolchain ] ───▶ [ QEMU Virt Platform ] ───▶ [ Console Output ]

🚀 Quick Start
Prerequisites
Rust / Cargo (Latest version recommended)
QEMU for RISC-V (qemu-system-riscv64)
xPack RISC-V GCC Toolchain (riscv-none-elf-gcc / g++)
Installation & Build
Bash
# 1. Clone Repository

git clone https://github.com/your-username/riscv-tool.git

cd riscv-tool

# 2-A. Build in Lite mode (C-only, ultra-lightweight)

cargo build --release

# 2-B. Build in Full mode (Supports C++ and stdio.h)

cargo build --release --features full

💻 Usage
1. Run C++ Example (Full Mode) Bash# Run C++ File

./target/release/riscv-tool run test_complex.cpp

test_complex.cpp Code Example: C++#include <stdio.h>

// 1. C++ Global Object Constructor Test

class GlobalLogger {

public:

GlobalLogger() {

printf("[Init] C++ Global Constructor Executed!\n");

}

};

GlobalLogger g_logger;

// 2. Virtual Functions and Polymorphism

class BaseDevice {

public:

virtual void show() { printf("Base Device\n"); }

};

class UARTDevice : public BaseDevice {

public:

void show() override { printf("SiFive UART0 (MMIO: 0x10000000)\n"); }

};

int main() { 
printf("Hello RISC-V C++ World!\n"); 

BaseDevice* dev = new UARTDevice(); 
dev->show(); // polymorphic behavior 

printf("Formatted Output - Hex: 0x%X, Dec: %d\n", 0xDEADBEEF, 2026); 
fflush(stdout); 

return 0;
}

🔧 Technical Details
Linker Script (link.ld) & Memory Map
RAM Start Address: 0x80000000 (QEMU virt default)
C++ Constructor Array: .init_array Sequential initialization of global objects via section mapping
Symbols Provided: __global_pointer$, __bss_start, _end (Newlib _sbrk for heap memory management)
System Call Stubs (c_runtime_header)
_write: Serialized output of characters to UART0 MMIO registers.
_sbrk: Dynamically allocates a memory region starting from the _end symbol as heap memory.
exit_qemu: Stops the QEMU emulator session by writing the value 0x5555 to address 0x100000.
📜 License
This project is licensed under the MIT License - see the LICENSE file for details.
