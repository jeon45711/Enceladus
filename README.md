# 🚀 RISC-V Bare-metal Dual-Mode Toolchain (Enceladus)

**Enceladus** is an ultra-lightweight execution and development toolchain for RISC-V (RV64GC) targets. 

It seamlessly handles both **ultra-lightweight bare-metal C** and **feature-rich C++** (with Standard C++ / Newlib `stdio.h` support) environments within a single codebase using feature toggling and automatic file extension detection.

---

## ✨ Key Features

- **⚡ Dual-Mode Architecture**:
  - **Lite Mode**: Ultra-lightweight bare-metal C development without standard libraries (`-nostdlib`).
  - **Full Mode**: Full Newlib C library (`stdio.h`) and standard C++ runtime support.

- **🔄 Automatic Compiler Selection**:
  - `.c` source files -> `riscv-none-elf-gcc`
  - `.cpp` / `.cc` source files -> `riscv-none-elf-g++`

- **🛠️ Embedded C++ Support**:
  - Global constructor/destructor execution (handles `.init_array` linker sections and `__libc_init_array` runtime).
  - Full support for virtual functions and polymorphism (vtable).
  - Dynamic memory allocation (`new` / `delete`) backed by custom `_sbrk` stubs.

- **🖥️ Seamless QEMU Emulation**:
  - Custom system call stubs optimized for the QEMU `virt` platform (UART0 IO register: `0x10000000`).
  - Clean emulator session termination via SiFive Test Finisher.

---

## 🏗️ System Architecture

```text
  [ User Source ] (.c / .cpp)
         │
         ▼
  [ Compiler Engine ]
    ├── Extension Detection (.cpp -> g++, .c -> gcc)
    ├── Feature Check (Lite vs Full)
    └── Dynamic Code Injection (c_runtime_header + link.ld)
         │
         ▼
  [ RISC-V Toolchain ] ───▶ [ QEMU Virt Platform ] ───▶ [ Console Output ]

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
