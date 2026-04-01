# Interrupts

An interrupt is a signal sent by hardware or software to the CPU to indicate that an event requires immediate attention.

Instead of continuously checking devices for events (polling), the CPU pauses its current execution and transfers control to a special routine that handles the event.

Common examples include:

- keyboard input
- mouse movement
- network packet arrival
- timer events

Interrupts improve efficiency because the CPU can perform other work until a device needs attention.

## Interrupt controller and IRQ

Hardware devices send an electrical signal to an **interrupt controller**. The interrupt controller manages multiple interrupt sources and forwards them to the processor.

Each interrupt source is associated with a unique **IRQ (Interrupt Request) number**. The operating system uses this number to determine which device triggered the interrupt.

Example:

- timer interrupt
- disk interrupt
- network interrupt

## Interrupt handler

An **interrupt handler** (also called an Interrupt Service Routine or ISR) is a function executed when an interrupt occurs.

Its responsibilities typically include:

- identifying the source of the interrupt
- performing minimal required processing
- notifying the rest of the system that work needs to be done

Interrupt handlers run in a special **interrupt context**, which means:

- they must execute quickly
- they cannot block or sleep
- they usually avoid heavy computation

Because of these restrictions, interrupt handlers only perform critical work and defer the rest.

## Top half and bottom half

To keep interrupt handling efficient, work is often divided into two parts.

**Top half**

The top half executes immediately when the interrupt occurs. It performs time critical tasks such as acknowledging the interrupt or reading device status.

**Bottom half**

The bottom half performs deferred processing that can run later. This allows the system to resume normal execution quickly.

This design prevents interrupts from blocking the CPU for long periods.

## Deferred work mechanisms

Operating systems provide mechanisms to execute the bottom half later.

Common techniques include:

**Softirqs**

A low level kernel mechanism used for high performance interrupt processing. Multiple softirqs can run simultaneously on different processors.

**Tasklets**

Built on top of softirqs and easier to use. Tasklets ensure that the same tasklet never runs concurrently on multiple processors.

**Work queues**

Work queues run tasks inside kernel threads. Since they run in process context, they can sleep and perform blocking operations.

In practice:

- softirqs are used for highly performance critical subsystems
- tasklets are commonly used for deferred interrupt work
- work queues are used when the deferred task may block

## Interrupt context

When an interrupt handler runs, the CPU executes in **interrupt context**.

Key characteristics:

- there is no associated user process
- handlers cannot sleep
- handlers must complete quickly

Because of these constraints, interrupt handlers typically perform minimal work and delegate complex processing to deferred mechanisms.

## Key takeaways

Interrupts allow hardware devices to notify the CPU when attention is needed, avoiding inefficient polling.

An interrupt triggers an interrupt handler that executes in interrupt context. Since handlers must run quickly and cannot block, most systems split interrupt processing into a fast **top half** and a deferred **bottom half** using mechanisms such as tasklets or work queues.