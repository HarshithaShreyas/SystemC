GIT Credentials: Harshitha1698  Harshi@6964
ChatGpt : lharshitha7891@gmail.com  Harshi@123
OLAT : sys28nax@rptu.de  Shantha@123@

---------------------------------------------///SystemC Basics/// CHAPTER 2 -------------------------------------------------------------

TOPICS DISCUSSED: SC_MODULE, SC_CTOR, SC_HASPROCESS, delta delay (enables the simulation of concurrency in a sequential simulator), stages of SystemC Simulation Kernel, SC_METHOD, SC_THREAD, sc_event, sc_time, dynamic sensitivity
 --For combinational logic or processes that do not need to maintain state, SC_METHOD is typically used. For sequential logic or processes that need to wait for events, SC_THREAD is typically used
 --No wait() statements with SC_METHOD - will crash simulation
 --In SystemC, SC_THREAD processes do not require a sensitivity list because they are designed to be self-scheduling. This means that they control their own execution using wait() statements.
 --SC_METHOD processes are event-driven and use a sensitivity list to specify which events will trigger them.
 --Events are caused or fired through the event class member function notify(): - should ideally be done with a time argument 
 --in case of multiple notifications, only the first one is noted, rest are ignored
 --to trigger events more than once, sc_event can't be used, sc_event_queue has to be used
--Wall Clock Time and Simulation Time
--wait() statements in SC_THREAD are examples of dynamic sensitivity
-- WE CAN’T BIND PORTS TO PORTS DIRECTLY!! this will lead to problems that we will see in the next lecture - signals are required to interact with SystemC Kernel - at the edges, bindings are allowed without a signal as well because these are special ports that themselves act as signals
RELEVANT EXAMPLES: delta_delay(SR-LATCH), feedback_loop(delta cycle running forever), swapping_example, thread_example, sc_event_and_queue(sc_event), clock_generator(sc_hasprocess & vcd file), not_chain(not and topmodule)

--------------------------------------------///SystemC Advanced CHAPTER 3--------------------------------------------------------------

TOPICS DISCUSSED: data types, and_reduce() methods,sc_lv and sc_bv, Channels, Interfaces, Primitive Channel  - {FIFO, Buffer, Signal}, Dynamic Processes, Report Handling, Callbacks

--signal is special case of Channel - Channels are containers for communication protocols
--Interface defines channel behaviour, Channel implements one or more interfaces
--Specialized Ports such as sc_in, sc_out, sc_inout are used for sc_signal, for RTL modelling and easy use
--sc_signal is a special type of channel - so are sc_fifo and sc_buffer, both being special cases of sc_signal which implement notifications, along with read and write
--signals can be exchanged with a FIFO or a Buffer - because the interface of all these signals and FIFOs is the same, we can interchange the communication models - an example of this is to put in a Ethernet Network or CAN bus or SPI Bus or I2C - modules do not know which channel is there because they use the same interface for write and read
-- sc_in<int> foo; is short MACRO for sc_port< sc_signal_in_if<int> > Foo;
--sc_out<bool> bar; which is short MACRO for sc_port< sc_signal_inout_if<bool> > Bar;
--sc_port has the parameters - Interface (required), N (maximum number of channels to be bound - optional), Policy (optional)
--complex channels like FIFO can only be modeled using SC_THREADS because SC_METHODS can't be used for blocking processes - blocking functions call an internal wait(), which SC_METHOD can't allow (will result in runtime error)
--primitive channels in SystemC = sc_signal, sc_buffer, sc_fifo, sc_mutex, sc_semaphore use sc_prim_channel
--making your own custom primitve channel --> see slide 33 or example 'custom_signal'
--Hierachical Channels (non-primitive, active) --> have logic + usual channel stuff (useful in buses like CAN which use Arbitration)
--sc_buffer:==> The difference to sc_signal is that with sc_buffer an event is generated each time the write() method is called.With sc_signal, an event is only generated if the old and the new value of the signal are different
--! Two Ports cannot be bound to the same channel - for signals and FIFOs in example ‘portarrays’ - doing so gives an error - sc_fifo<T> cannot have more than one writer
--In SystemC, the order of execution of processes within the same delta cycle is not deterministic.
--If ports are created dynamically, use port.size() within module's process function to work on inidividual ports iteratively
--4 kinds of callbacks for sc_module, sc_prim_channel, sc_port and sc_export - see slide 53
◼ before_end_of_elaboration()
◼ end_of_elaboration()
◼ start_of_simulation()
◼ end_of_simulation()

RELEVANT EXAMPLES: datatypes,custom_fifo(Custom Channel Implementation),nb_fifo(nb_read and nb_write) fifo_example, kpn_example, mutex_example, custom_signal(primitive update mechanism), portarrays, multiports, sc_event_and_queue, event_finder, dynamic_processes, reporting(report handling), callbacks


--------------------------------------------///TLM Basics/// CHAPTER 4--------------------------------------------------------------------
In TLM, we are not simulating clocks and pins, just the function calls between components - by transfer of packets - from the event perspective, 
the only thing that matters is we know when communication starts and when does it end.

TOPICS DISCUSSED: speed v/s accuracy tradeoff, LT vs AT difference, LT, Generic Payload, Sockets, Time Annotation, Interconnect Component, Temporal Decoupling, Quantum Keeper, DMI, Debug Transport

--Everything in TLM domain is memory mapped
--no need to deal with delta cycling
--In TLM, we will use Call by Reference using a pointer - in TLM, we don’t transfer the entire packet, we just transfer a pointer (a 64 bit number) - makes simulation really fast
--LT faster than AT {as fast as possible, fast enough to build SW, SW dev centric}
--LT implements blocking transport calls
--can't be used to model out-of-order execution
--Processes can run ahead of simulation time (temporal decoupling)

--In nonstandard TLM transaction between producer consumer without interconnect, the consumer does not need a port because it directly implements the transport call defined in transactionInterface.

TARGET Functions : class tlm_fw_transport_if<…>
virtual … b_transport(…) = 0;
virtual … nb_transport_fw(…) = 0;
virtual … get_direct_mem_ptr(…) = 0;
virtual … transport_dbg(…) = 0;
Initiator Functions:
class tlm_bw_transport_if<…>
virtual … nb_transport_bw(…) = 0;
virtual … invalidate_direct_mem_ptr(…) = 0;
--Binding Initiator and Targets ==> ports of both derived from sc_port so we bind the iSocket to tSocket
--The key idea of timing annotation is that the recipient is obliged to behave as if it had received the transaction at time sc_time_stamp() + delay
--in LT style, we want to avoid wait() statements as much as possible - instead, try to annotate time on the local variables which are passed with the transport calls and the target can add its own custom delay to it
--If we have multiple transport calls, make the local delay variable keep accumulating the value of delay{pass it as argument to every transport call}, 
and then wait with the accumulated delay value wait(delay) once at the end of a sequence of transports

--Interconnect acts as a Target on Initiator (Producer) side and a Initiator on the Target (Consumer) side
--If there is one initiator and 2 Targets, the Interconnect internally will have two iSockets and one tSocket (because of above)
--needs to be derived from both forward and backward transport interfaces
--Pipelined Transactions or multiple initiator calls concurrently cannot be modelled with LT as transport calls are atomic, so there can be no collisions

--Temporal Decoupling trades accuracy for speed, by reducing number of context switches - process keeps control until end of Quantum - lessens context switches in case of multiple initiators
--Synchronization in Quantum Keeper ==> if there is time left in the quantum, it is consumed by internally calling wait() so that quantum notion is not broken.
--In case of overrun, starts the next transaction with an offset equal to the delay encountered
--smaller the quantum, higher the accuracy, larger the quantum, higher the speed

--DMI used to bypass a detailed Interconnect (probably defined in RTL) - further increase in simulation speed
--Target can set a DMI hint, but before Initiator can know about it, atleast one transaction has to be performed
--Target can even invalidate DMI regions, meaning it can specify certain regions of memory where DMI must not be used.

--Debug Transport used to bypass bootloading process - putting bootloader into memory should not be simulated - so start simulation with everything loaded, thus no delay
--when we use Debug Transport, it does not simulate time, so bootloading process can be done in it, and it won't be accounted towards simulation time.

RELEVANT EXAMPLES: custom_tlm(Basic TLM-->consumer as hierarchical channel), tlm_lt_initiator_target, tlm_lt_initiator_interconnect_target, tlm_quantum_keeper, tlm_lt_dmi



--------------------------------------------///TLM AT/// CHAPTER 5--------------------------------------------------------------------

Base Protocol 1: Using BW PATH(4 Handshake - BEGIN_REQ;REQ_END;BEGIN_RESP;END_RESP)
Base Protocol 2: Using Return Path (BEGIN_REQ;TLM ACCEPTED /END_REQ;BEGIN_RESP;END_RESP)
Base Protocol 3: Skip END_REQ (BEGIN_REQ;BEGIN_RESP;END_RESP)
Base Protocol 4: Early Completion (BEGIN_REQ;TLM_Completed)

