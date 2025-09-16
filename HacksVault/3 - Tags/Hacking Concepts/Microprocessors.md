# What is a Microprocessor

Microprocessors, the (electronically) pulsating heart of our computers and digital devices, are not infallible. They can contain flaws or `vulnerabilities` that attackers can exploit to perform unauthorised actions within a computer system. These vulnerabilities range from simple design flaws to complex security loopholes.

In some cases, these vulnerabilities are unintentional, arising from oversights or errors in the design process. In other instances, they may be deliberately introduced as `backdoors` to allow certain users unauthorised access. Regardless of their origin, all microprocessor vulnerabilities pose a risk to the security and integrity of our digital systems.

A `microprocessor` is an integrated circuit (`IC`) that encapsulates the functions of a `central processing unit` (`CPU`) on a single silicon chip. A microprocessor is a type of CPU, but not all CPUs are microprocessors. For example, the CPU in a mainframe computer is typically implemented using multiple ICs, while the CPU in a personal computer is typically implemented using a single microprocessor. Its role is to fetch, decode, and execute instructions, facilitating various computational and control tasks.

The functional architecture of a microprocessor revolves around several essential components, including the `control unit` (`CU`), the `arithmetic logic unit` (`ALU`), and an `Instruction Set Architecture` (`ISA`), among others.

- The `control unit` is part of a computer's CPU that directs the operation of the processor. It tells the computer's memory, arithmetic/logic unit and input and output devices how to respond to the instructions that have been sent to the processor.
    
- The `arithmetic logic unit` is a major component of the computer system's central processing unit (CPU), or the 'brain'. It performs arithmetic and logic operations, which are fundamental functions of computers.
    
- The `Instruction Set Architecture` is an abstract model of a computer that defines the supported data types, the register set, what memory addressing modes are available, and the instruction set or the set of machine-language commands that the computer can understand and execute.
    

`Transistors`, the fundamental building blocks of a microprocessor, play a crucial role in its operation. These tiny electronic switches can turn the flow of electricity `on` or `off`, representing `binary states`, i.e., `1s` and `0s`. In the `fetch-decode-execute cycle`, transistors function to store and manipulate the binary data during the execution of instructions. The `control unit` uses combinations of these binary values to represent different commands and data. The `arithmetic logic unit` uses transistors based on these binary instructions to perform arithmetic and logical operations. The more transistors a microprocessor contains, the more instructions it can process, enhancing its speed and performance.

## Microprocessor Design

Microprocessor design is a multifaceted discipline that incorporates several stages, each of which can bear upon the potential vulnerability of the resulting system. Amongst others, these stages include `architectural design`, `logic design`, `circuit design`, `physical design`, and `verification`,

### Architectural Design

`Architectural design`, the first phase of microprocessor design, refers to the formulation of the processor's architectural specifications, which involves deciding the instruction set architecture. The design choices made in this phase significantly impact the microprocessor's performance, power efficiency, and cost. For instance, selecting a CISC (Complex Instruction Set Computer) or RISC (Reduced Instruction Set Computer) architecture can dramatically influence the complexity and speed of the processor. In short, CISC is like a Swiss Army Knife with many tools in one package, while RISC is like a specialist tool designed to do one thing very well.

### Logic Design

Next is the `logic design` stage, where the specifications defined in the architectural design are translated into concrete logic operations. Here, digital logic principles and Boolean algebra are used to create a detailed schematic of `logic gates`. This process includes `creating the data path` (which performs arithmetic and logical operations) and `control units` (which orchestrate data flow). The end result is a `register-transfer level` (`RTL`) description that serves as a blueprint for circuit design.

### Circuit Design

`Circuit design` entails converting the RTL into specific electronic circuits. This involves choosing circuit-level implementations for each logic gate using `transistors`, `resistors`, and `capacitors`. The goal is to optimise circuits for factors like speed, power consumption, and silicon area. For instance, an `AND` gate can be implemented using two transistors in a series configuration.

### Physical Design

The `physical design` stage involves defining the spatial layout of components on the silicon chip. This process includes placement (deciding where each component should go) and routing (determining the paths for electrical connectivity). Here, factors like heat dissipation and power distribution need careful consideration because they significantly impact chip reliability and performance.

### Verification

The last phase is `verification`, a crucial step to ensure the microprocessor performs as intended. Formal methods are used to prove or disprove the correctness of a system against a certain specification. This includes static timing analysis to ensure all logic functions correctly within the defined clock cycle time. Verification helps identify and rectify potential issues before mass fabrication, saving time and cost.

It should be noted that while the design process has the potential to introduce vulnerabilities, it also provides opportunities for embedding security features into the microprocessor.

## Microprocessor Optimization Strategies

`Microprocessor Optimization Strategies` are the various methods and techniques to enhance the performance and efficiency of a microprocessor. These strategies focus on improving processing speed, reducing power consumption, and enhancing other crucial aspects like instruction execution and data handling.

### Pipelining

`Pipelining` is a crucial strategy used to `improve the throughput of instruction execution`. It involves breaking down the execution of instructions into discrete `stages` that can be `processed simultaneously`. Each instruction is fed through the pipeline stage-by-stage sequentially, akin to an assembly line in a manufacturing plant.

The main stages of an instruction pipeline typically include `instruction fetch`, `instruction decode`, `execute`, `memory access`, and `write back`. By allowing multiple instructions to be at different stages of execution simultaneously, pipelining significantly increases the instruction throughput and utilisation of the processor's resources.

### Speculative Execution

`Speculative Execution` is an optimisation method used to boost processing speed. This technique is based on `making educated guesses` about the potential direction a program might take, particularly when faced with a conditional branch instruction - a juncture where the program can follow different paths based on a specific condition.

Instead of waiting to determine the correct path (which can cause delays), the microprocessor will predict the most likely path and execute instructions along that pathway. If this prediction is correct, it results in substantial time savings and increases overall performance. However, this method requires an efficient rollback mechanism. If the prediction is incorrect, the microprocessor must discard the speculatively executed instructions, revert to its pre-speculation state, and resume along the correct path.

### Caching

`Caching` is a technique employed to speed up memory access. As fetching data from the main memory is time-consuming, microprocessors use caches, which are `small`, `high-speed memory units` placed between the CPU and the main memory. They store `frequently or recently accessed data`, enabling quicker retrieval when needed.

Caches are usually organised in a hierarchy of levels (`L1`, `L2`, `L3`), with `L1` being the smallest and fastest and directly interfacing with the CPU, while the `lower levels are larger and slower`. Caching is crucial for performance but can also be a source of security vulnerabilities. Cache-based side-channel attacks, for instance, deduce sensitive data by observing timing differences in memory access, which are influenced by the state of the cache.
# Microprocessor Vulnerabilities

Microprocessor vulnerabilities refer to the weaknesses or flaws in the design or implementation of microprocessors that can be exploited to compromise the security of a computing system. These vulnerabilities can manifest across various layers of the microprocessor's operational stack, from its hardware foundations to its firmware interfaces and the software it executes.

One widely recognised class of microprocessor vulnerabilities pertains to `side-channel attacks`. These `exploit indirect information leakages`, such as `timing information`, `power consumption`, or `electromagnetic emissions`. An infamous example of a side-channel attack is the `Spectre` and `Meltdown` vulnerabilities discovered in 2018.

Microprocessor vulnerabilities represent a significant concern due to these components’ critical role in modern computational infrastructure, from personal devices and embedded systems to servers and data centres. Their study and mitigation have become important areas of focus within cybersecurity.

## Spectre

Spectre is a class of microprocessor vulnerabilities that was publicly disclosed in 2018. Officially identified as `CVE-2017-5753` (`bounds check bypass`, Spectre-V1) and `CVE-2017-5715` (`branch target injection`, Spectre-V2).

Spectre was independently discovered and reported by Jann Horn from Google Project Zero and Paul Kocher. Kocher collaborated with several others, including Daniel Genkin, Mike Hamburg, Moritz Lipp, and Yuval Yarom.

Spectre is unique in that it breaks the isolation between different applications, allowing an attacker to trick error-free programs into leaking their secrets. `Spectre` takes advantage of the `speculative execution` technique used in modern microprocessors.

In a normal scenario, if the processor's predictions are accurate, the speculatively executed instructions are committed, and their results are used. If the predictions are wrong, the instructions and their direct effects are discarded to maintain program correctness. Herein lies the catch. Despite discarding the direct effects of incorrect speculative execution, subtle, indirect effects may persist, particularly in micro-architectural structures like the `cache`.

The `Spectre` vulnerability intentionally causes the processor to make incorrect predictions which initiate the speculative execution of a specially chosen set of instructions. These instructions are designed to deliberately modify the state of the processor in a way that wouldn't happen under normal conditions. For example, they could cause specific data to be loaded into the cache that wouldn't usually be there.

Although these manipulated instructions and their immediate outcomes are discarded when the processor detects the incorrect prediction, the modifications made to the cache remain.

However, attackers still cannot directly access this data; they need to use a side-channel attack to infer what data has been loaded into the cache, revealing potentially sensitive data, which should have been secure and inaccessible.

The Spectre vulnerability thus represents a potent exploit that leverages a fundamental performance feature of modern microprocessors, turning it into a channel for information leakage.

The most immediate consequence of Spectre is its potential to compromise the security of computing systems. By exploiting speculative execution, Spectre enables an attacker to read sensitive information from the memory of other programs running on the same system, including potential passwords, encryption keys, and other confidential data. This represents a severe violation of the fundamental security principle of process isolation.

## Meltdown

Meltdown, officially identified as `CVE-2017-5754`, is a severe microprocessor vulnerability that was publicly disclosed in 2018, alongside the aforementioned Spectre vulnerabilities.

Meltdown was independently discovered and reported by three teams: Jann Horn from Google Project Zero; Werner Haas and Thomas Prescher from Cyberus Technology; and Daniel Gruss, Moritz Lipp, Stefan Mangard, and Michael Schwarz from Graz University of Technology.

Unlike Spectre, which `breaks the isolation between different applications`, Meltdown dissolves the more fundamental `isolation between user applications and the operating system`. This allows a malicious program to access the memory of other programs and the operating system, potentially gaining access to sensitive information.

The key to the Meltdown vulnerability is its exploitation of a feature of modern microprocessors known as `out-of-order execution`. This is a performance-enhancing technique in which the processor `executes instructions not in the sequential order` in which they appear in the program but in an `order dictated by the availability` of input data and execution units, thereby maximising resource utilisation and throughput. However, while the final results of execution are committed in order, the effects of instructions executed out-of-order can still be observed in the processor's micro-architectural state, even if these instructions are later rolled back.

Meltdown leverages this characteristic in a specific way. It begins by inducing an exception — for example, by attempting to access a privileged memory location that is off-limits to user programs. While handling this exception would usually involve discarding the effects of the offending instruction, the out-of-order execution allows further instructions that depend on this illegal access to be executed before the exception is handled.

For instance, one such instruction could load data from the privileged memory location (referenced indirectly via the exception-causing instruction) into the cache. Even though the exception is subsequently handled and the effects of the offending instruction are discarded, the changes to the cache remain. Then by using a side-channel attack — specifically, a cache timing attack — the attacker can infer the data that was loaded into the cache and, as a result, the contents of the privileged memory location.

This vulnerability is critical because it `undermines kernel/user space isolation`, a `cornerstone of operating system security`.

Like Spectre, Meltdown illustrates the potential security risks that can arise from the pursuit of performance in microprocessor design and the complex interplay between hardware and software in modern computing systems.

Meltdown's most immediate and severe impact is its potential to compromise the security of computing systems. Meltdown breaks the fundamental security boundary between user applications and the operating system kernel, potentially allowing a malicious application to read sensitive kernel memory, including personal data, passwords, and cryptographic keys.

## Spectre vs Meltdown

Both Spectre and Meltdown are critical vulnerabilities in microprocessors that exploit performance-enhancing features of processor architecture, specifically speculative execution and out-of-order execution. While they share certain characteristics, there are notable differences in their mechanics and potential impacts.

Spectre and Meltdown exploit different aspects of modern processor design. Spectre (`CVE-2017-5753` and `CVE-2017-5715`) capitalises on `speculative execution`.

Meltdown (`CVE-2017-5754`), on the other hand, exploits `out-of-order execution`.

`Spectre breaks the isolation between different applications`, allowing an attacker to trick error-free programs into leaking their secrets, potentially leading to cross-process, virtual machine, or sandbox escape attacks. It is more difficult to exploit than Meltdown but also harder to mitigate completely.

Conversely, `Meltdown breaks the isolation between user applications and the operating system`, allowing a program to access the memory of other programs and the operating system. This can lead to severe violations of system integrity and confidentiality. Meltdown is easier to mitigate with techniques like Kernel Page Table Isolation (KPTI), but these mitigations can carry significant performance overheads.

## Mitigation strategies

Mitigation strategies for Spectre and Meltdown have been a subject of intense research and development since their disclosure. Both hardware and software-based solutions have been proposed and implemented.

The primary objective of these mitigation techniques is to prevent unauthorised access to sensitive data by exploiting the vulnerabilities without significantly compromising system performance; a task easier said than done.

### Retpoline

`Retpoline` is an effective mitigation technique for `Spectre`. Replacing potentially hazardous `indirect software branches` prevents the speculative execution that Spectre exploits.

An `indirect software branch` refers to a type of program instruction that guides the execution path based on specific conditions. Unlike a `direct branch`, where the destination is pre-determined and known in advance, the destination of an indirect branch is determined dynamically during runtime.

The fundamental idea behind `retpoline` is to modify the control flow so that `speculative execution is avoided` when encountering indirect branches.

Instead of using traditional branch instructions that attackers can manipulate to leak sensitive information, retpolines rely on a sequence of instructions that redirect the program flow without allowing speculative execution to take place. However, `retpolines` may impose a performance overhead, particularly in applications with many indirect branches.

### Compiler Barriers

Changes to how compilers generate code can also help mitigate `Spectre`. For instance, by introducing specific code constructs or instructions known as `barriers`.

`Memory Barriers` (or `Fence Instructions`) ensure that all load and store memory operations before the barrier are completed before any operations that come after the barrier. They're used to maintain consistency and prevent undesired reordering of memory accesses by the compiler or processor.

`Branch Prediction Barriers` are used to inhibit speculative execution at certain points in the code where it could lead to a security vulnerability. By limiting branch prediction and speculative execution, they help reduce the potential for malicious activities.

The purpose of these barriers is to enforce strict control over the flow of execution, `disallowing the processor from executing instructions speculatively`. Compilers can safeguard critical code sections against Spectre attacks by strategically adding these barriers.

### KPTI

`KPTI` is the primary mitigation technique against `Meltdown`. It isolates the kernel's page table from the page tables of user space processes to prevent any potential information leakage from kernel memory. However, this technique can introduce performance overheads due to increased context switch times.

### Microcode updates

CPU manufacturers have issued `Microcode updates` to enable CPUs to implement more fine-grained control and restrictions on speculative execution. This allows the operating system to apply stricter measures and prevent the exploitation of `Meltdown`-like vulnerabilities at the hardware level.

The Spectre and Meltdown vulnerabilities have brought forth a critical juncture in microprocessor design, highlighting the inextricable link between high-performance computing and security. These vulnerabilities, hinged on the performance optimisation features that define modern microprocessors, have uncovered the inherent security risks in present designs.

While effective, the mitigation techniques employed thus far, including `retpoline`, `compiler modifications`, `Kernel Page Table Isolation`, and `microcode updates`, have demonstrated the limitations and trade-offs that must be navigated to secure systems. While they mitigate the immediate risks of Spectre and Meltdown, they do not entirely eliminate them nor prevent the emergence of new variants.