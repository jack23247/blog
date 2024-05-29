# Demistifying the AS/400

It's easy to look at IBM's AS/400 and say that is the quintessential "black box" of computer systems, that works in mysterious and impenetrable ways, so far removed from the concepts we're all familiar with (mainly thanks to the prevalence of UNIX in the ).

## A little bit of history

The AS/400 is categorized as a *midrange* system, which is IBM parlance for the space formerly occupied by minicomputers; it descends from the System/3 line of computers that were originally sold to small institutions that could not afford even the smallest S/360s (and/or didn't even need them). 

The System/3 was introduced in 1969 and had a very lean architecture, optimized for cost and ran a small operating system called System Control Program (SCP), which could be loaded from cards or from disk. Programs for the system were written in RPG II and Operator Control Language (OCL) was provided for job control. The S/3 was programmed using punched card decks, but some models provided an operator console, which could be used to start and stop jobs. Despite its limitations, the S/3 proved quite popular due to its lower cost compared to the smallest offerings in the S/360 line, and can be considered a spiritual successor to the 1401.

In 1975, the S/3 was succeeded by the System/32: a smaller and fully integrated system which contained a printer, a console, storage and the CPU in a desk-like enclosure. The S/32's CPU was called Control Storage Processor (CSP) and was a 16-bit microcoded design that emulated the S/3 instruction set. The S/32 ran an updated version of SCP, which was partly reimplemented in microcode for performance reasons, and introduced an editor called Source Entry Utility (SEU) that allowed the operator to write programs directly on the console.

In 1977, IBM introduced the System/34: a successor to the S/32 that could support multiple users connected via block-mode terminals using the 5250 protocol, introduced with this machine. The S/34 retained the CSP, which was used to perform I/O operations[^1] and added a second processor, called Main Storage Processor (MSP), which ran user programs.

A simplified diagram of the S/34 architecture is available [here](http://bitsavers.informatik.uni-stuttgart.de/pdf/ibm/system34/fe/SY31-0458-3_System_34_5340_System_Unit_Theory_Diagrams_Manual_Jul79/SY31-0458-3_Section_01_Introduction.pdf).


[^1] IBM systems often included a programmable storage processor: the S/360 line of mainframes, for example, achieved high throughput by employing multiple Channel Processors (CPs) that ran independently of one another, allowing the main CPU to keep working while slow I/O operations were performed. This technique is similar to Direct Memory Access (DMA), which is commonplace, but allows for much more complex operations to be performed entirely without the intervention of the CPU.