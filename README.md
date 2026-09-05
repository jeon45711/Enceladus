🚀 RISC-V Bare-metal Dual-Mode Toolchain (Enceladus)Enceladus is an ultra-lightweight execution and development toolchain for RISC-V (RV64GC) targets.It seamlessly handles both ultra-lightweight bare-metal C and feature-rich C++ (with Standard C++ / Newlib stdio.h support) environments within a single codebase using feature toggling and automatic file extension detection.
✨ Key Features
⚡ Dual-Mode Architecture:Lite Mode: Ultra-lightweight bare-metal C development without standard libraries (-nostdlib).Full Mode: Full Newlib C library (stdio.h) and standard C++ runtime support.
🔄 Automatic Compiler Selection:.c source files $\rightarrow$ riscv-none-elf-gcc.cpp / .cc source files $\rightarrow$ riscv-none-elf-g++
🛠️ Embedded C++ Support:Global constructor/destructor execution (handles .init_array linker sections and __libc_init_array runtime).Full support for virtual functions and polymorphism (vtable).Dynamic memory allocation (new / delete) backed by custom _sbrk stubs.
🖥️ Seamless QEMU Emulation:Custom system call stubs optimized for the QEMU virt platform (UART0 IO register: 0x10000000).Clean emulator session termination via SiFive Test Finisher.
         │
         ▼
  [ Compiler Engine ]
    ├── Extension Detection (.cpp -> g++, .c -> gcc)
    ├── Feature Check (Lite vs Full)
    └── Dynamic Code Injection (c_runtime_header + link.ld)
         │
         ▼
  [ RISC-V Toolchain ] ───▶ [ QEMU Virt Platform ] ───▶ [ Console Output ]
🚀 Quick StartPrerequisitesRust / Cargo (Latest stable version recommended)QEMU for RISC-V (qemu-system-riscv64)xPack RISC-V GCC Toolchain (riscv-none-elf-gcc / g++)Installation & BuildBash# 1. Clone the repository
git clone https://github.com/jeon45711/Enceladus.git
cd Enceladus

# 2-A. Build for Lite Mode (C-only, ultra-lightweight)
cargo build --release

# 2-B. Build for Full Mode (C++ and stdio.h support)
cargo build --release --features full
💻 UsageRunning a C++ Program (Full Mode)Bash# Execute a C++ source file
./target/release/enceladus run test_complex.cpp
test_complex.cpp Example:C++#include <stdio.h>

// 1. Testing C++ Global Constructor
class GlobalLogger {
public:
    GlobalLogger() {
        printf("[Init] C++ Global Constructor Executed!\n");
    }
};

GlobalLogger g_logger;

// 2. Polymorphism and Virtual Functions
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
    dev->show(); // Dispatches to UARTDevice::show()

    printf("Formatted Output - Hex: 0x%X, Dec: %d\n", 0xDEADBEEF, 2026);
    fflush(stdout);
    
    return 0;
}
🔧 Technical DetailsLinker Script (link.ld) & Memory MapRAM Start Address: 0x80000000 (QEMU virt default).
C++ Constructor Array: Maps .init_array section to sequentially invoke global object constructors.Provided Symbols: Defines __global_pointer$, __bss_start, and _end for Newlib _sbrk heap memory management.
System Call Stubs (c_runtime_header)_write: Directs serial character output directly to UART0 MMIO registers.
_sbrk: Allocates dynamic memory starting from the _end symbol boundary.exit_qemu: Writes 0x5555 to 0x100000 to terminate the QEMU emulator session gracefully

.📜 LicenseThis project is licensed under the MIT License - see the LICENSE file for details.
