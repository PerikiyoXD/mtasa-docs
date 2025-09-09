# MTA Coding Guidelines

## Magic Numbers

**Don't:** Use raw numbers without explanation
```cpp
return *(float*)0xC81308; // What is this?
```

**Do:** Define with descriptive names
```cpp
constexpr std::uintptr_t NUM_WETROADS = 0xC81308;
return *(float*)NUM_WETROADS;
```

**Prefixes for definitions:**
```cpp
constexpr std::uintptr_t FUNC_RemoveRef          = 0x4C4BB0;  // Function addresses
constexpr std::uintptr_t ARRAY_aCannons          = 0xC80740;  // Array addresses  
constexpr std::uintptr_t STRUCT_CAESoundManager  = 0xB62CB0;  // Struct addresses
constexpr std::size_t    SIZE_CWaterCannon       = 0x3CC;     // Object sizes
constexpr std::size_t    NUM_CWaterCannon_Offset = 0x32C;     // Numbers/offsets
constexpr std::uintptr_t CLASS_CText             = 0xC1B340;  // Class addresses
constexpr std::uintptr_t VAR_CTempColModels      = 0x968DF0;  // Variable addresses
```

## Naming Conventions

**Variables and types:** `lowerCamelCase`
```cpp
SSomeStruct valueOne;
ESomeEnum   valueTwo;
```

**Functions and classes:** `UpperCamelCase`
```cpp
void DoSomething();
class MyClass;
```

**Class members:** `m_` prefix
```cpp
CVector m_vecPosition;
bool    m_isVisible;
std::int32_t m_playerCount;
```

**Avoid Hungarian notation in new code:**
```cpp
// Old style (only for consistency with existing code)
float         fValue;
unsigned char m_ucValue;
bool          g_bCrashTwiceAnHour;

// New style
float         value;
std::uint8_t  m_value;
bool          g_crashTwiceAnHour;
```

## Functions

**Don't:** Use `void` for empty parameters
```cpp
void MyFunction(void);
MyFunction(void);
```

**Do:** Use empty parentheses
```cpp
void MyFunction();
MyFunction();
```

## Code Structure

### Early Returns
**Don't:** Nest conditions deeply
```cpp
bool RespawnObject(CElement* pElement)
{
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

**Do:** Exit early, reduce nesting
```cpp
bool RespawnObject(CElement* pElement)
{
    if (!IS_OBJECT(pElement))
        return false;
    
    auto* pObject = static_cast<CObject*>(pElement);
    if (!pObject)
        return false;
        
    pObject->Respawn();
    return true;
}
```

### Early Continue
**Don't:** Nest loop conditions
```cpp
for (auto i = 0; i < 255; i++) {
    if (conditionA) {
        someCode();
        if (conditionB) {
            otherCode();
        }
    }
}
```

**Do:** Use continue to flatten logic
```cpp
for (auto i = 0; i < 255; i++) {
    if (!conditionA)
        continue;
        
    someCode();
    if (!conditionB)
        continue;
        
    otherCode();
}
```

### Auto Usage
**Use `auto*` for obvious pointer types:**
```cpp
// Instead of this verbose line
CDeathMatchObject* pObject = static_cast<CDeathmatchObject*>(pEntity);

// Use this
auto* pObject = static_cast<CDeathmatchObject*>(pEntity);
```

### Ternary Operators
**Use for simple conditions:**
```cpp
// Don't
const CPositionRotationAnimation* CObject::GetMoveAnimation()
{
    if (IsMoving()) {
        return m_pMoveAnimation;
    } else {
        return nullptr;
    }
}

// Do
const CPositionRotationAnimation* CObject::GetMoveAnimation()
{
    return IsMoving() ? m_pMoveAnimation : nullptr;
}
```

### Braces for Single Statements
**Omit braces for short single statements:**
```cpp
// Short condition
if (!isStillRunning)
    StopMoving();

// Add blank line after braceless statements for readability
PreCheck();
if (!isStillRunning)
    StopMoving();

PostCheck();
```

**Keep braces for multi-line or complex statements:**
```cpp
// Don't do this
for (auto i = 0; i < count; ++i)
    for (auto j = 0; j < size; ++j)
        for (auto k = 0; k < depth; ++k)
            ComplexOperation();
```

### Simple Getters in Headers
**Place simple return functions in headers:**
```cpp
// In header file
std::int32_t GetGameSpeed() const noexcept { return m_gameSpeed; }

// Don't put in .cpp file unless complex logic needed
```

### Remove Unnecessary Parentheses
```cpp
// Don't
bool CClientPed::IsDead()
{
    return (m_status == STATUS_DEAD);
}

// Do
bool CClientPed::IsDead()
{
    return m_status == STATUS_DEAD;
}
```

### Use Range-Based Loops
```cpp
std::vector<std::int32_t> vec;
std::map<std::string, std::int32_t> playerScores;

// Don't
for (std::vector<std::int32_t>::iterator it = vec.begin(); it != vec.end(); ++it)

// Do
for (const auto& value : vec)
for (const std::int32_t& value : vec)

// For maps with structured binding
for (const auto& [name, score] : playerScores)

// When iterator needed, use auto
for (auto it = vec.begin(); it != vec.end(); ++it)
```

## Header Files

**Always include copyright and pragma:**
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

**Use `#pragma once` instead of include guards** (remove old `#ifndef` guards when updating)

## Source Files

**Always include copyright and Standard Include (except when not needed):**
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

### Type Casting
**Use C++ casts instead of C-style casts:**
```cpp
// Don't
auto* pObject = (CObject*)pElement;
float value = (float)intValue;
std::uintptr_t address = (std::uintptr_t)pointer;

// Do
auto* pObject = static_cast<CObject*>(pElement);
float value = static_cast<float>(intValue);
std::uintptr_t address = reinterpret_cast<std::uintptr_t>(pointer);

// For compile-time type punning
std::uint32_t bits = std::bit_cast<std::uint32_t>(floatValue);
```

**Cast types:**
- `static_cast` - Safe conversions (numeric, class hierarchy)
- `reinterpret_cast` - Pointer/reference type changes  
- `const_cast` - Remove const/volatile (avoid when possible)
- `dynamic_cast` - Runtime polymorphic casting
- `std::bit_cast` - Type punning

### Specifiers
**Use `const` and `constexpr` everywhere possible:**
```cpp
const std::int32_t maxPlayers = 32;
constexpr float PI = 3.14159f;
```

**Use `noexcept` when functions don't throw (but be careful - throws terminate program):**
```cpp
std::int32_t GetPlayerCount() const noexcept { return m_playerCount; }
```

### Null Pointers
```cpp
// Don't
ptr = NULL;

// Do  
ptr = nullptr;
```

### Member Initialization Lists
**Don't:** Initialize in constructor body
```cpp
CObject::CObject(CElement* pParent, CObjectManager* pManager)
{
    m_type = CElement::OBJECT;
    m_pObjectManager = pManager;
    m_model = 0xFFFF;
    m_pMoveAnimation = NULL;
}
```

**Do:** Use initialization lists
```cpp
CObject::CObject(CElement* pParent, CObjectManager* pManager)
    : m_type(CElement::OBJECT),
      m_pObjectManager(pManager),
      m_model(0xFFFF),
      m_pMoveAnimation(nullptr)
{
    SetTypeName("object");
    pManager->AddToList(this);
}
```

### Standard Types
**Use namespaced standard types:**
```cpp
// Don't
uint32_t value;
size_t length;

// Do
std::uint32_t value;  // from <cstdint>
std::size_t length;   // from <cstddef>
```

**Use `std::string` instead of `SString` in new code:**
```cpp
// Don't (in new code)
SString playerName;

// Do
std::string playerName;
```

## Argument Parser

**New functions:**
```cpp
{"pathListDir", ArgumentParser<pathListDir>},
```

**Legacy functions (for backward compatibility):**
```cpp
{"outputChatBox", ArgumentParserWarn<false, OutputChatBox>},
```

More info: [Lua API Wiki](https://github.com/multitheftauto/mtasa-blue/wiki/Lua-API#how-to-add-a-new-lua-definition)
