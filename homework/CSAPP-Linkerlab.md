relocation.cc

```c++
#include "relocation.h"
#include <sys/mman.h>
#include <iostream>
#include <vector>

// this script is for Task0 and Task1

void handleRela(std::vector<ObjectFile> &allObject, ObjectFile &mergedObject, bool isPIE)
{
    /* When there is more than 1 objects, 
     * you need to adjust the offset of each RelocEntry
     */
    /* Your code here */
    if (allObject.size() > 1)
    {
        uint64_t sum = 0;
        for (auto &object : allObject) 
        {
            for(auto &reloc : object.relocTable)
            {
                reloc.offset = reloc.offset + sum;
            }
            sum += object.sections[".text"].size;
        }
    }
    

    /* in PIE executables, user code starts at 0xe9 by .text section */
    /* in non-PIE executables, user code starts at 0xe6 by .text section */
    uint64_t userCodeStart = isPIE ? 0xe9 : 0xe6;
    uint64_t textOff = mergedObject.sections[".text"].off + userCodeStart;
    uint64_t textAddr = mergedObject.sections[".text"].addr + userCodeStart;

    /* Your code here */
    for (auto &obj : allObject)
    {
        for (auto &re : obj.relocTable) // relocEntry
        {
            uint64_t symbolAddr = re.sym->value;
            uint64_t baseAddr = reinterpret_cast<uint64_t>(mergedObject.baseAddr);
            uint64_t targetAddress = baseAddr + textOff + re.offset;

            // if (re.type == 2)
            // {
            //     int valueToFill = symbolAddr - (textAddr + re.offset) + re.addend;
            //     *reinterpret_cast<int*>(targetAddress) = valueToFill;
            // }

            if (re.type == R_X86_64_32) // elf.h giving alias as Number, but I prefer Signal itself
            {
                // R_X86_64_32 uses absolute addressing
                int valueToFill = symbolAddr + re.addend;
                *reinterpret_cast<int*>(targetAddress) = valueToFill;
            }
            else if (re.type == R_X86_64_PLT32 || re.type == R_X86_64_PC32)
            {
                // R_X86_64_PLT32 and R_X86_64_PC32 use PC-relative addressing
                int valueToFill = symbolAddr - (textAddr + re.offset) + re.addend;
                *reinterpret_cast<int*>(targetAddress) = valueToFill;
            }
            else {
                std::cerr << "Fail! " << re.type << std::endl;
                abort();
            }
        }
    }
}
```

resolve.cc

```c++
#include "resolve.h"
#include <iostream>
#include <vector>

#define FOUND_ALL_DEF 0
#define MULTI_DEF 1
#define NO_DEF 2

// this script is for Task2, Task3, Task4, Task5

std::string errSymName;

int callResolveSymbols(std::vector<ObjectFile> &allObjects);

void resolveSymbols(std::vector<ObjectFile> &allObjects) {
    int ret = callResolveSymbols(allObjects);
    if (ret == MULTI_DEF) {
        std::cerr << "multiple definition for symbol " << errSymName << std::endl;
        abort();
    } else if (ret == NO_DEF) {
        std::cerr << "undefined reference for symbol " << errSymName << std::endl;
        abort();
    }
}

/* bind each undefined reference (reloc entry) to the exact valid symbol table entry
 * Throw correct errors when a reference is not bound to definition,
 * or there is more than one definition.
 */
int callResolveSymbols(std::vector<ObjectFile> &allObjects)
{
    /* Your code here */
    // if found multiple definition, set the errSymName to problematic symbol name and return MULTIDEF;
    // if no definition is found, set the errSymName to problematic symbol name and return NODEF;
    std::unordered_map<std::string, Symbol*> StrongMap;
    std::unordered_map<std::string, Symbol*> WeakMap;
  
    /*
    Strong:
        - Mul-Def: ERROR! (task3)
        - ok: add into mapping
    
    Weak:
        - Strong Exists: attach to Strong (task4)
        - No Strong: All use itself (task5)

    No Signal:
        - Error! (task2)
    */

    // Multi-Def
    for (auto &obj : allObjects) 
    {
        for (auto &sym : obj.symbolTable) // symbol
        {
            if (sym.bind == STB_GLOBAL && sym.index != SHN_UNDEF && sym.index != SHN_COMMON)  
            {
                // Strong & Global itself, need to consider Multi-Conflicts
                if (StrongMap.find(sym.name) != StrongMap.end()) 
                {
                    // conflict
                    errSymName = sym.name;
                    return MULTI_DEF;
                }
                // no conflict, then add current mapping
                StrongMap[sym.name] = &sym;
            }
            else if (sym.bind == STB_GLOBAL && sym.index == SHN_COMMON) 
            {
                // Global but Weak, put into WeakClass at first
                WeakMap[sym.name] = &sym;
            }
        }
    }

    // Attach weak symbols to strong symbols if they exist
    for (auto &pair : WeakMap) 
    {
        auto symName = pair.first;

        if (StrongMap.find(symName) != StrongMap.end()) {
            // find StrongOne, then attach
            pair.second->value = StrongMap[symName]->value;
            pair.second->index = StrongMap[symName]->index;
        }
    }

    // Now both Strong and Weak can be utilized

    // None-Def
    for (auto &obj : allObjects) 
    {
        for (auto &re : obj.relocTable) // symbol
        {
            std::string symName = re.sym->name;
            
            if (StrongMap.find(symName) != StrongMap.end()) {
                re.sym = StrongMap[symName];
            }
            else if (WeakMap.find(symName) != WeakMap.end()) {
                re.sym = WeakMap[symName];
            }
            else {
                errSymName = symName;
                return NO_DEF;
            }
        }
    }

    return FOUND_ALL_DEF;
}

```

>- completed by CS2201H_BoxuanHu
>- Time Taken: 5h

