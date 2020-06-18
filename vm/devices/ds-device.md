# Nabla Devices

## Register Key

Devices that are used by the Nabla VM will key off of register 10. At the end of an instruction cycle, if a device sees information regarding its operation in r10 it will act accordingly. 

### What does this mean ? 

Don't use registers 10 - 15 without a clear goal. They are reserved for device operations

## Devices


| Device     | ID      | Description                 |  
|---         |---      |---                          |  
|    IO      | 0x0A    | User and disk io operations |
| Network    | 0x0B    | TCP/UDP Network operations  |
| Host       | 0x0C    | Host operations             |
| DataStore  | 0x0D    | Data Store operations       |

## Data Store Device

Data Store trigger register and state structure.

           ID         SUB-ID    [ ----------------------- Varies by SUB-ID  ----------------------- ]
        0000 1101 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 
        
**ID**     - The ID of the Data Store Device (0x0D)

SUB-IDs: 

| Sub Id          | VALUE | Description                    |  
|---              |---    |---                             |  
|    allocate     |  0    | Allocate storage for something |
|    free         |  1    | Free allocated storage         |
|    copy         |  5    | Copy data                      |
|    store        |  10   | Store some data                |
|    load         |  20   | Load some data                 |
|    size         |  30   | Retrieve data size             |
|    reset        |  50   | Clear all data from memory     |

**Overview**

The Data Store device allows us to offload the allocating and deallocating of data without requiring permanent residence on the systen's global stack. 

**allocate**

* Register 10

           ID         SUB-ID    [------------------ BYTES --------------------] [------ UNUSED -----]
        0000 1101 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

If the allocation request can be executed, a 1 will be in r11, otherwise a 0 will be in r11. The ID given to the request
will be in r12. 

**free**

* Register 10

           ID         SUB-ID    [-------------------------- UNUSED ---------------------------------]
        0000 1101 | 0000 0001 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

* Register 11

        [---------------------------------------- DATA ID ------------------------------------------]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

**copy**

* Register 10

           ID         SUB-ID    [-------------------------- UNUSED ---------------------------------]
        0000 1101 | 0000 0101 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

* Register 11

        [------------------------------------- DATA ID FROM ----------------------------------------]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

* Register 12

        [------------------------------------- DATA ID TO ------------------------------------------]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

The destination data size will be updated to whatever is required for data to fit. It might be smaller, or larger than the initial allocation. 

If the copy request could be executed, a 1 will be in r11, otherwise a 0 will be in r11. The only reason a copy should fail
is if one of the given IDS does not exist.

**store**

* Register 10

           ID         SUB-ID    [-------------------------- UNUSED ---------------------------------]
        0000 1101 | 0000 1010 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

* Register 11

        [------------------------------------- DATA ID TO ------------------------------------------]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

* Register 12

        [ --------- GLOBAL STACK START ADDRESS ----- ]  [ --------- GLOBAL STACK END ADDRESS ----- ]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

Stores data from the global stack starting at START ADDRESS and ending at END ADDRESS (not including end address). 
If the request could be executed, a 0 will be in r11, otherwise a 1 will be in r11.

If the destination space can not accomodate the data, stack end address < stack start address, or if the
id does not exist this operation will fail

**load**

* Register 10

           ID         SUB-ID    LOAD TYPE   [--------------------- UNUSED --------------------------]
        0000 1101 | 0000 1010 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

Load Types: 

| Load Type       | VALUE | Description                       |  
|---              |---    |---                                |  
|    Specific     |  0    | Interpret r12 as two GS addresses |
|    Dump         |  1    | Ignore r12 push data onto GS      |

If the request could be executed, a 0 will be in r11, otherwise a 1 will be in r11.

* Register 11

        [------------------------------------- DATA ID FROM ----------------------------------------]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

**load:Specific**

* Register 12

        [ --------- GLOBAL STACK START ADDRESS ----- ]  [ --------- GLOBAL STACK END ADDRESS ----- ]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

Specific loading will fail in the event the range is not large enough to accomodate the data stored at the given address, 
the start address is greater than the end address, or the given address is not found

**load:Dump**

Whatever is in the index stored in the address in r11 will be pushed onto global stack. Upto the caller
to determine size and what to do with the data.

**size**

* Register 10

           ID         SUB-ID    [-------------------------- UNUSED ---------------------------------]
        0001 1110 | 0011 0010  | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

* Register 11

        [------------------------------------- DATA ID ---------------------------------------------]
        0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 


If the request could be executed, a 0 will be in r11, otherwise a 1 will be in r11.
Result in r12

**reset**

* Register 10

           ID         SUB-ID    [-------------------------- UNUSED ---------------------------------]
        0000 1101 | 0011 0010  | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 | 0000 0000 

This command will not fail in a way that the user can handle.