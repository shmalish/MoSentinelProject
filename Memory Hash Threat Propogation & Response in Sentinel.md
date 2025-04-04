# Memory Hash Threat Propogation & Response in Sentinel

The first aim of this project is to learn how memory works in Linux and Windows more in depth. While talking to my good friend about my understanding on how an elf gets maipulated and handled in memory, I came to find that I didn't know much about how it worked. Therefore this project will also be tailored in a way that it teaches while implementing. 

The second aim of this project is to understand how Sentinel works because companies glaze Microsoft which is fair because no one uses Libre Office. Therefore, this creates room for academic contribution that might be useful albeit probably not significant since I barely know how Sentinel works while writing this. 

## Project idea

Memory stores executables and it may be possible to hash the executables that are running in memory. While you can just hash binaries from a /bin/ folder, from what I understand they get manipulated in memory i.e sections expand and data gets changed. 

The project will first find every executable running in memory, find the start and end i.e elf headers in linux and essentially create a whitelist of allowed applications to run in memory. Therefore if a rogue process is identified, the hash wont match in the allow list and therefore an alert can be triggered. This will be done via either an agent scan or a memory snapshot which both have their pros and cons. Agent scans will be more efficient since you won't need to do a memory dump of every device within an organisation however, if coded horribly it will slow down peoples devices which isn't practical for an organisation. Memory snapshots will be easier to analyse ,however, for an organisation and SOC team it isn't practical to take a snapshot of 50,100,300+ devices and have some sort of process that analyses them. I don't know which route I will take yet because I don't understand memory yet. Another caveat here is that you will probably see lots of false positives due to variable changes etc etc.  

After a hash is identified that is not on the allow list, an alert will be trigged and will be sent to Microsoft Sentinel. This allows the SOC team to investigate as well as potentially check to see if any other devices contain the same hash in memory. This could therefore indicate there is malware running across an organisation OR an adobe update. 

## How do programs get executed in Linux?

The elf format is the most popular binary format in linux. It always contains a program header table near the start of the file. This table is an array of structures each discribing a segment or other information the system needs to prepare for execution. 

The function load_elf_binary() first checks to see if the file is an elf. If not it will throw an error or exception. It will then go through program header entries to find PT_INTERP and whether the stack should be executable (This is found from PT_GNU_STACK entry). It then does some complex fancy stuff which I don't understand. 

The elf header contains meta data about the executable i.e whether it is 32 or 64 bit, endianess, version and architecture. Table 1 contains a list of fields that are inside the elf header and provides an explanation. 

The program header table stores information about segments where each segment is made up of at least one section.This table tells the kernel how to create the process and how to map segments into memory. When running a program, the kernal loads the elf header and program header table into memory. It then loads the contents that are specified in LOAD in the program header table into memory and checks if the interpreter is needed.

The section header tables store information about sections which is used during dynamic linking. 

Section text holds the instructions that the program needs to run, .rodata and .data contain actual, initialised data which the program will need in memory. The memory reserves more space for the data segment than specified in the ELF file to make room over uninitialised variables.

![](/home/ubuntu/snap/marktext/9/.config/marktext/images/2025-04-04-17-37-49-image.png)

| Field                             | Explanation                                                                                                                                                                            |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Magic                             | These are the first bytes in the ELF header. They identify the file <br>as an ELF and contain information that processors can use to interpret <br>the file.                           |
| Class                             | The value in the class field indicates the architecture of the file. As such the ELF can either be 32-bit or 64-bit.                                                                   |
| Data                              | This field specifies the data encoding. This is important to help <br>processors interpret incoming instructions. The most common data <br>encodings are little-endian and big-endian. |
| Version                           | Identifies the ELF file version (set to 1)                                                                                                                                             |
| OS/ABI                            | ABI is short for Application Binary Interface. In this case, it <br>defines how functions and data structures can be accessed in the <br>program.                                      |
| ABI Version                       | This field specifies the ABI version.                                                                                                                                                  |
| Type                              | The value in this field specifies the object file type. For <br>instance, 2 is for an executable, 3 is for a shared object, and 4 is for<br> a core file.                              |
| Machine                           | This specifies the architecture needed for the file.                                                                                                                                   |
| Version                           | Identifies the object file version.                                                                                                                                                    |
| Entry point address               | This indicates the address where the program should start executing.<br> In the case that the file is not an executable file, the value in this <br>field is set to 0.                 |
| Start of program headers          | This is the offset on the file where the program headers start.                                                                                                                        |
| Start of section headers          | This is an offset that indicates where the section headers start.                                                                                                                      |
| Flags                             | This contains flags for the file.                                                                                                                                                      |
| Size of this header               | This specifies how big the ELF header is.                                                                                                                                              |
| Size of program header            | The value in this field specifies how big an individual program header is.                                                                                                             |
| Number of program headers         | This indicates how many program headers there are.                                                                                                                                     |
| Size of section headers           | The value in this field shows how big an individual section header is.                                                                                                                 |
| Number of section headers         | This indicates how many section headers there are.                                                                                                                                     |
| Section header string table index | The section table index of the entry representing the section name string table                                                                                                        |

### The elf layout in memory





# References

[How programs get run: ELF binaries [LWN.net]](https://lwn.net/Articles/631631/?utm_source=chatgpt.com)[How programs get run: ELF binaries [LWN.net]](https://lwn.net/Articles/631631/?utm_source=chatgpt.com)

[Program Header](https://www.sco.com/developers/gabi/latest/ch5.pheader.html)

https://chessman7.substack.com/p/understanding-elf-file-layout-in?utm_medium=web
