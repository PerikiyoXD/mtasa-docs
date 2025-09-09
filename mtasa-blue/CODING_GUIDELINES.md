# MTA Coding Guidelines

## Magic Numbers

**Don't:** Use raw numbers without explanation
```cpp
return *(float*)0xC81308; // What is this?
```

**Do:** Define with descriptive names
```cpp
#define NUM_WETROADS 0xC81308
return *(float*)NUM_WETROADS;
```

**Prefixes for definitions:**
```cpp
#define FUNC_RemoveRef          0x4C4BB0  // Function addresses
#define ARRAY_aCannons          0xC80740  // Array addresses  
#define STRUCT_CAESoundManager  0xB62CB0  // Struct addresses
#define SIZE_CWaterCannon       0x3CC     // Object sizes
#define NUM_CWaterCannon_Offset 0x32C     // Numbers/offsets
#define CLASS_CText             0xC1B340  // Class addresses
#define VAR_CTempColModels      0x968DF0  // Variable addresses
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
int     m_playerCount;
```

**Avoid Hungarian notation in new code:**
```cpp
// Old style (only for consistency with existing code)
float         fValue;
unsigned char m_ucValue;
bool          g_bCrashTwiceAnHour;

// New style
float         value;
unsigned char m_value;
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

**Do:** Exit early, reduce nesting
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

### Early Continue
**Don't:** Nest loop conditions
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

**Do:** Use continue to flatten logic
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
const CPositionRotationAnimation* CObject::GetMoveAnimation() {
    if (IsMoving()) {
        return m_pMoveAnimation;
    } else {
        return nullptr;
    }
}

// Do
const CPositionRotationAnimation* CObject::GetMoveAnimation() {
    return IsMoving() ? m_pMoveAnimation : nullptr;
}
```

### Braces for Single Statements
**Omit braces for short single statements:**
```cpp
// Short condition
if (!bStillRunning)
    StopMoving();

// Add blank line after braceless statements for readability
PreCheck();
if (!bStillRunning)
    StopMoving();

PostCheck();
```

**Keep braces for multi-line or complex statements:**
```cpp
// Don't do this
for (dont)
    for (do)
        for (this)
            complexOperation();
```

### Simple Getters in Headers
**Place simple return functions in headers:**
```cpp
// In header file
int GetGameSpeed() const noexcept { return m_iGameSpeed; }

// Don't put in .cpp file unless complex logic needed
```

### Remove Unnecessary Parentheses
```cpp
// Don't
bool CClientPed::IsDead() {
    return (m_status == STATUS_DEAD);
}

// Do
bool CClientPed::IsDead() {
    return m_status == STATUS_DEAD;
}
```

### Use Range-Based Loops
```cpp
std::vector<int> vec;
std::map<std::string, int> myMap;

// Don't
for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); it++)

// Do
for (const auto& v : vec)
for (const int& v : vec)

// For maps with structured binding
for (const auto& [key, value] : myMap)

// When iterator needed, use auto
for (auto it = vec.begin(); it != vec.end(); it++)
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

## Modern C++ Practices

### Specifiers
**Use `const` and `constexpr` everywhere possible:**
```cpp
const int maxPlayers = 32;
constexpr float PI = 3.14159f;
```

**Use `noexcept` when functions don't throw (but be careful - throws terminate program):**
```cpp
int GetPlayerCount() const noexcept { return m_playerCount; }
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
CObject::CObject(CElement* pParent, CObjectManager* pManager) {
    m_iType = CElement::OBJECT;
    m_pObjectManager = pManager;
    m_usModel = 0xFFFF;
    m_pMoveAnimation = NULL;
}
```

**Do:** Use initialization lists
```cpp
CObject::CObject(CElement* pParent, CObjectManager* pManager)
    : m_iType(CElement::OBJECT),
      m_pObjectManager(pManager),
      m_usModel(0xFFFF),
      m_pMoveAnimation(nullptr) {
    
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
