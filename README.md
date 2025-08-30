# pio_irq library
Raspberry Pico library that supports handling PIO interrupts with C++ objects.  
Goal: let the PIO state machines call the very C++ object that can manage (or watch) their particular event.

It's most useful in:  
- object oriented designs, where
- PIO state machines are managed by objects. 
- When you have an object that needs to know when a state machine has completed an activity.
- When you have multiple state machines, all  independently performing their task, and your program needs to know what's happening.  

A typical design is to have your own little object (a handler) for each state machine + interrupt# combination.
You register that object in the library. It will get called when that state machine fires that interrupt.

The core of this library is a single class, ```pio_irq```. It is specialised in configuring PIO interrupt handling, and routing any interrupts to the object that can handle it (handler objects). 
- It knows how to inform a PIO state machine that we're interested in a particular PIO interrupt.  
- It knows what object in your firmware has the duty to react on that interrupt on that PIO state machine. When an interrupt occurs, it will call that object.

There is only a single requirement for the handler class(es): The ```operator()``` has to be available (overloaded).

## documentation:
[1: usage and example](https://community.element14.com/products/raspberry-pi/b/blog/posts/oo-library-to-handle-pico-pio-relative-interrupts)  
[2: library design](https://community.element14.com/products/raspberry-pi/b/blog/posts/oo-library-to-handle-pico-pio-relative-interrupts-library-design)  

## Example: all available PIO state machines generate an interrupt:  
Check [usage and example](https://community.element14.com/products/raspberry-pi/b/blog/posts/oo-library-to-handle-pico-pio-relative-interrupts) for the complete example code.   
```cpp
import pio_irq;
import your_handler;

const uint32_t IRQ_NUM = 0U; // we handle interrupt # 0 (the one the PIO program fires)
const uint32_t IRQ_CHAN = 0U; // in this example, we use PIO interrupt channel 0

struct your_handler {
    inline void operator()() {
        // you received an interrupt. do stuff
    }
};

// the objects that will handle PIO interrupts. Each will react to a single state machine's interrupt number 0
// for this example, let's use all the available state machines
// if using more than 4, you may have to increase PICO_MAX_SHARED_IRQ_HANDLERS
// do this by adding the following line to the CMake file:
// add_compile_definitions(PICO_MAX_SHARED_IRQ_HANDLERS=12)

using handler_t = your_handler;
std::array<handler_t, 4 * NUM_PIOS> handlers {{
      {pio0, 0}, {pio0, 1}, {pio0, 2}, {pio0, 3}
    , {pio1, 0}, {pio1, 1}, {pio1, 2}, {pio1, 3}
#if (NUM_PIOS > 2) // pico 2
    , {pio2, 0}, {pio2, 1}, {pio2, 2}, {pio2, 3}
#endif
}};

// the PIO interrupt manager
using pio_irq_manager_t = pio_irq::pio_irq<handler_t, IRQ_NUM>;

int main() {
    // program PIO, init the used state machines, do other init things
    // then:

    
    for (auto &h: handlers) {
        pio_irq_manager_t::register_interrupt(IRQ_CHAN, h.pio, h.sm, true);
    }

    for (auto &h: handlers) {
        pio_irq_manager_t::register_handler(h.pio, h.sm, &h, true);
    }

    // enable the state machines
```

## Adding the lib to your Raspberry Pico project
The repository has its own makefile. you add it by fetching from it in your ```CMakeFiles.txt```. It is then available to your code as library stepper.
```cmake
# your CMakeFiles.txt
# ... 

include(FetchContent)
FetchContent_Declare(pio_irq
  GIT_REPOSITORY "https://github.com/jancumps/pio_irq.git"
  GIT_TAG "origin/main"
)
FetchContent_MakeAvailable(pio_irq)

# ...

# add stepper as a library to your executable
add_executable(your_project)
# ...
target_link_libraries(your_project
        pico_stdlib
        pio_irq
)
```
That is all it takes to integrate this design in your project.

## Other project considerations  
On a RP2350 (Pico2), the SDK default setting for ```PICO_MAX_SHARED_IRQ_HANDLERS``` limits the number of interrupt/callback enabled motors you can use. When you have more than 4 interrupt/callback enabled motors in your project, override the SDK setting in your ```CMakeFiles.txt``` with the number of motors in your code. Add it before Pico SDK import.
```
add_compile_definitions(PICO_MAX_SHARED_IRQ_HANDLERS=10)
```


## demo project that uses this lib: 
[advanced example with notification](https://community.element14.com/products/raspberry-pi/b/blog/posts/raspberry-pio-stepper-library-documentation---2-advanced-example-with-notification)  
[usage and example](https://community.element14.com/products/raspberry-pi/b/blog/posts/oo-library-to-handle-pico-pio-relative-interrupts)  

## toolchain requirements: 
- CMake 3.28 or higher
- GCC 15.1 or higher
- Pico C SDK 2.1.1
- tested with Pico 1, Pico-W, Pico2, Pico2-W (ARM and RISC-V)