# MTA Coding Guidelines

## Magic Numbers
**Don't:**
```cpp
return *(float*)0xC81308; // What is this?
```

**Do:**
```cpp
#define NUM_WETROADS 0xC81308
return *(float*)NUM_WETROADS;
```

**Prefixes:**
- `FUNC_` - Function addresses
- `ARRAY_` - Array addresses  
- `STRUCT_` - Struct addresses
- `SIZE_` - Object sizes
- `NUM_` - Numbers/offsets
- `CLASS_` - Class addresses
- `VAR_` - Variable addresses

## Naming Conventions
- **Variables/types**: `lowerCamelCase`
- **Functions/classes**: `UpperCamelCase`  
- **Members**: `m_` prefix
- **No Hungarian notation** in new code

```cpp
// Good
CVector m_vecPosition;
void DoSomething();
SSomeStruct valueOne;

// Bad (unless maintaining old code)
float fValue;
unsigned char m_ucValue;
```

## Functions
**Don't:**
```cpp
void MyFunction(void);
```

**Do:**
```cpp
void MyFunction();
```

## Code Structure

### Early Returns
**Don't:**
```cpp
bool RespawnObject(CElement* pElement) {
    if (IS_OBJECT(pElement)) {
        CObject* pObject = static_cast<CObject*>(pElement);
        if (pObject) {
            pObject->Respawn();
            return true;
        }
    }
    return false;
}
```

**Do:**
```cpp
bool RespawnObject(CElement* pElement) {
    if (!IS_OBJECT(pElement))
        return false;
    
    CObject* pObject = static_cast<CObject*>(pElement);
    if (!pObject)
        return false;
        
    pObject->Respawn();
    return true;
}
```

### Loops
**Don't:**
```cpp
for(auto i = 0; i < 255; i++) {
    if(conditionA) {
        someCode();
        if(conditionB) {
            otherCode();
        }
    }
}
```

**Do:**
```cpp
for(auto i = 0; i < 255; i++) {
    if (!conditionA)
        continue;
        
    someCode();
    if (!conditionB)
        continue;
        
    otherCode();
}
```

### Other Rules
- Use `auto*` for pointers when type is obvious
- Ternary operators for simple conditions
- Omit braces for single-line statements (add blank line after)
- Simple getters go in headers: `int GetSpeed() const noexcept { return m_speed; }`
- No unnecessary parentheses
- Use range-based loops: `for (const auto& item : container)`

## Headers
**Always start with:**
```cpp
/*****************************************************************************
 *
 *  PROJECT:     Multi Theft Auto
 *  LICENSE:     See LICENSE in the top level directory
 *
 *  Multi Theft Auto is available from https://www.multitheftauto.com/
 *
 *****************************************************************************/

#pragma once
```

## Sources
**Always start with:**
```cpp
/*****************************************************************************
 *
 *  PROJECT:     Multi Theft Auto
 *  LICENSE:     See LICENSE in the top level directory
 *
 *  Multi Theft Auto is available from https://www.multitheftauto.com/
 *
 *****************************************************************************/

#include "StdInc.h"
```

## Modern C++ Practices

### Specifiers
- Use `const`/`constexpr` everywhere possible
- Use `noexcept` (but be careful - throws terminate program)

### Pointers
**Don't:**
```cpp
ptr = NULL;
```

**Do:**
```cpp
ptr = nullptr;
```

### Initialization
**Don't:**
```cpp
CObject::CObject() {
    m_iType = CElement::OBJECT;
    m_pManager = pManager;
    m_usModel = 0xFFFF;
}
```

**Do:**
```cpp
CObject::CObject()
    : m_iType(CElement::OBJECT),
      m_pManager(pManager),
      m_usModel(0xFFFF) {
}
```

### Types
- Use `std::uint32_t` instead of `uint32_t`
- Use `std::string` instead of `SString` in new code
- Include `<cstdint>`, `<cstddef>` for standard types

## Argument Parser
**New functions:**
```cpp
{"pathListDir", ArgumentParser<pathListDir>},
```

**Old functions (to maintain compatibility):**
```cpp
{"outputChatBox", ArgumentParserWarn<false, OutputChatBox>},
```
