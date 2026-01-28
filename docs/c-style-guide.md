# C Coding Style Guide

This document defines the coding standards and guidelines for C code in this project. All code must follow these rules to ensure consistency, safety, and maintainability.

---

## 1. NAMING & TYPES (MUST)

### Typedefs
- **UPPERCASE_WITH_UNDERSCORES**: All typedefs must use uppercase with underscores
- Defined in `global.h` (e.g., `UINT32`, `UINT16`)

### Struct Typedefs
- End in `_STRUCT`
- Tag name (if used) matches typedef
- All uppercase with underscores
- Example:
  ```c
  typedef struct USER_DATA_STRUCT {
      UINT32 userId;
      char name[64];
  } USER_DATA_STRUCT;
  ```

### Union Typedefs
- End in `_UNION`
- Uppercase with underscores
- Example:
  ```c
  typedef union CONFIG_VALUE_UNION {
      UINT32 intValue;
      float floatValue;
  } CONFIG_VALUE_UNION;
  ```

### Enum Typedefs
- End in `_ENUM`
- Enumerators uppercase with underscores
- Non-zero starts allowed
- Example:
  ```c
  typedef enum STATUS_CODE_ENUM {
      STATUS_OK = 0,
      STATUS_ERROR = 1,
      STATUS_PENDING = 2
  } STATUS_CODE_ENUM;
  ```

### Constants & Macros
- **UPPERCASE_WITH_UNDERSCORES**
- Example:
  ```c
  #define MAX_BUFFER_SIZE 1024
  #define DEFAULT_TIMEOUT 30
  ```

### Variables
- **Local variables**: lowerCamelCase (e.g., `userId`, `bufferSize`)
- **Global/module variables**: CamelCase (e.g., `GlobalConfig`, `ModuleState`)
- **Minimize globals**: Avoid global variables when possible
- **Avoid underscores**: Use camelCase instead of snake_case for variables

### Physical Units
- Suffix variable names with physical units when applicable
- Examples:
  - `VolumeM3` (cubic meters)
  - `FlowRateKgHr` (kilograms per hour)
  - `TemperatureC` (Celsius)
  - `PressureKpa` (kilopascals)

### Booleans
- Use positive names that imply true/false
- Examples:
  - `isPathGood` (good: positive implication)
  - `isConnected`
  - `hasData`
  - ❌ `isPathBad` (bad: negative implication)

### Pointers
- Prefix with `p`, `ptr`, `P`, or `Ptr`
- Join `*` to the name
- Example:
  ```c
  UINT16 *pStatus;
  char *ptrBuffer;
  USER_DATA_STRUCT *pUserData;
  ```

### Database Values
- Variable names **must exactly match** the external label in `USM_DB_Defn`
- Case-sensitive matching is required

---

## 2. FUNCTIONS (MUST)

### Naming
- **Local functions**: lowerCamelCase (e.g., `parseUserData()`, `validateInput()`)
- **Global functions**: CamelCase with a unique module prefix
  - Format: `MODULE_FunctionName` or `MODULE_FunctionName` (optional underscore after prefix)
  - Examples: `BMEAS_Init()`, `BMEAS_ProcessData()`

### Function Definition Header
- Must include a description block
- Function name on its own line
- Parameters one per line
- **Inputs first**, then **outputs**
- Example:
  ```c
  /*
   * Description: Initializes the measurement module with the given configuration
   * Returns: STATUS_OK on success, error code on failure
   */
  STATUS_CODE_ENUM
  BMEAS_Init(
      const CONFIG_STRUCT *pConfig,    // Input: configuration parameters
      UINT32 timeout,                   // Input: timeout in seconds
      STATUS_STRUCT *pStatus            // Output: initialization status
  )
  {
      // Function body
  }
  ```

### Boolean-Returning Functions
- Names must imply TRUE/FALSE
- Header must state when TRUE and FALSE are returned
- Example:
  ```c
  /*
   * Description: Checks if the path is valid and accessible
   * Returns: TRUE if path is valid and accessible, FALSE otherwise
   */
  BOOL isPathValid(const char *pPath)
  {
      // Implementation
  }
  ```

### Returns
- Always declare return type (never implicit `int`)
- **Prefer a single return** at the end of the function
- Use goto for cleanup if necessary to maintain single return

---

## 3. HEADERS & FILES (MUST)

### Include Guards
- Use `#if !defined(...)` / `#define ...` / `#endif` pattern
- Or use `#ifndef` / `#define` / `#endif`
- Example:
  ```c
  #if !defined(MODULE_NAME_H)
  #define MODULE_NAME_H

  // Header content

  #endif // MODULE_NAME_H
  ```

### Module Headers
- Every `.c` file exposing globals must have a corresponding `.h`
- Headers should contain **only externally needed information** (information hiding principle)
- Internal implementation details stay in the `.c` file

### Global Definitions
- Ubiquitous items live in `global.h`
- Module-specific globals in module headers

### Source Files
- Naming: `lowerCamelCase.c`
- Include the standard file header banner with updated years and metadata
- Example:
  ```c
  /*
   * File: httpServerMain.c
   * Description: Main HTTP server implementation
   * Author: [Name]
   * Date: YYYY-MM-DD
   * Copyright: [Copyright info]
   */
  ```

---

## 4. STATEMENTS & COMMENTS (MUST)

### Casting
- Avoid unnecessary casts
- **Do not cast NULL** in comparisons (e.g., `ptr == NULL`, not `ptr == (TYPE *)NULL`)
- Cast NULL **only when passing** to a function that requires it

### Pointer Comparisons
- **Always compare explicitly to NULL**
- Example:
  ```c
  if (pBuffer != NULL) {  // Good
      // Use pBuffer
  }
  
  if (pBuffer) {  // Bad: implicit comparison
      // Use pBuffer
  }
  ```

### Line Length
- Maximum **140 characters** per line

### Comments

#### Major Block Headers
- Use for modules and functions
- Example:
  ```c
  /*
   * Module: User Management
   * Description: Handles user authentication and authorization
   */
  ```

#### Multi-line Comments
- Align with asterisks
- Example:
  ```c
  /*
   * This is a multi-line comment
   * with aligned asterisks
   */
  ```

#### Comment Content
- Focus on **"why"**, not **"how"**
- The code shows "how"; comments explain "why"
- Example:
  ```c
  // Good: explains why
  // Retry 3 times because the network may be temporarily unavailable
  
  // Bad: explains how (obvious from code)
  // Loop 3 times
  ```

#### End-line Comments
- Use only for:
  - Declarations
  - Block ends (e.g., `} // end if`, `#endif // MODULE_NAME_H`)

---

## 5. DEBUGGING (MUST)

### DEBUG Macros
- Use `DEBUGn(...)` macros: `DEBUG1`, `DEBUG2`, `DEBUG3`
- **Never delete** debug statements
- Disable via `#undef ENABLE_DEBUG`

### Format
- `DEBUGn(fn_name():<message>)`
- Example:
  ```c
  DEBUG1(parseUserData():Parsing user data for userId=%u, userId);
  DEBUG2(BMEAS_Init():Initialization complete);
  DEBUG3(validateInput():Input validation passed);
  ```

---

## 6. CONDITIONAL COMPILATION (MUST)

### Gas-Specific Code
- Use `#if defined` pattern
- Example:
  ```c
  #if defined METER_TYPE_GAS
      // Gas-specific code
  #endif
  ```

### Other Model-Specific Code
- Follow the same pattern for other conditional compilation
- Examples:
  ```c
  #if defined METER_TYPE_LIQUID
      // Liquid-specific code
  #endif
  
  #if defined EIGHT_PATH_MODEL
      // 8-path specific code
  #endif
  ```

---

## 7. system() USAGE (RESTRICT)

### Minimize Usage
- Avoid `system()` calls whenever possible
- Prefer direct API calls or library functions

### Required Checks When Used
When `system()` must be used, **always check**:

1. **WIFEXITED**: Check if process exited normally
2. **WEXITSTATUS**: Get exit status code
3. **WIFSIGNALED** / **WTERMSIG**: Check if terminated by signal
4. **WIFSTOPPED** / **WSTOPSIG**: Check if stopped

### Error Logging
- Log failures meaningfully
- Include what command was run and why it failed

### Example
```c
int status = system("command");
if (WIFEXITED(status)) {
    int exitStatus = WEXITSTATUS(status);
    if (exitStatus != 0) {
        DEBUG1(executeCommand():Command failed with exit status %d, exitStatus);
    }
} else if (WIFSIGNALED(status)) {
    DEBUG1(executeCommand():Command terminated by signal %d, WTERMSIG(status));
} else if (WIFSTOPPED(status)) {
    DEBUG1(executeCommand():Command stopped by signal %d, WSTOPSIG(status));
}
```

---

## 8. CYBERSECURITY & SAFE C (MUST)

### Input Validation
- **Validate all external inputs**:
  - CLI arguments
  - Environment variables
  - Sockets/pipes/files
  - Shared memory
  - Devices
  - Signals

### Strings & Arrays

#### Size Selection
- **Fixed-size** for:
  - Known sizes
  - Critical data
  - Data ≤ 512 bytes
- **Dynamic** otherwise

#### Bounded Functions
- Use bounded functions:
  - `strncpy()` instead of `strcpy()`
  - `fgets()` instead of `gets()`
  - `snprintf()` instead of `sprintf()`

#### strncpy() Caveat
- **May not NUL-terminate** if source is too long
- **Always terminate explicitly**:
  ```c
  strncpy(dest, src, sizeof(dest) - 1);
  dest[sizeof(dest) - 1] = '\0';  // Explicit NUL termination
  ```

### String Sanitization
- **Sanitize external strings** using whitelisting
- Reject any characters not in the allowed set

### Path Canonicalization
- Use `realpath()` to canonicalize user-provided paths
- Prevents path traversal attacks

### const Correctness
- Use `const` for:
  - String literals
  - Any by-reference parameter not modified
- Example:
  ```c
  void processData(const char *pInput, char *pOutput);
  ```

### Integer Safety

#### Prevent Overflow
- Unsigned wrap
- Signed overflow

#### Use size_t for Indices
- Array indices should be `size_t`

#### Pointer Arithmetic
- Keep within array bounds
- Example:
  ```c
  char buffer[100];
  char *ptr = buffer;
  // Ensure: ptr >= buffer && ptr < buffer + sizeof(buffer)
  ```

### Bitwise Operations

#### Rules
- **Only on unsigned types**
- **Never shift by negative** or ≥ bit width
- **Avoid mixing** bitwise and arithmetic on same data

### Memory Allocation

#### Pre-check Size Computations
- Check for overflow before allocation
- Example:
  ```c
  size_t count = n;
  size_t size = sizeof(TYPE);
  if (count > SIZE_MAX / size) {
      // Overflow would occur
      return ERROR;
  }
  void *ptr = malloc(count * size);
  ```

#### Dynamic Strings
- Prefer `asprintf()` for dynamic strings

#### Free Only Dynamic Memory
- Only call `free()` on dynamically allocated memory
- Never free stack variables or static data

#### Set Pointers to NULL
- After freeing, set reusable pointers to `NULL`
- Use `FREE()` macro:
  ```c
  #define FREE(ptr) do { free(ptr); (ptr) = NULL; } while(0)
  ```

### Formatted I/O

#### Valid Format Strings Only
- Ensure format strings match arguments

#### Never Use User Input as Format String
- **Critical security issue**: format string vulnerabilities
- Example:
  ```c
  // Bad: user input as format string
  sprintf(buffer, userInput);
  
  // Good: fixed format string
  sprintf(buffer, "%s", userInput);
  ```

### Files & TOCTOU (Time-of-Check-Time-of-Use)

#### Validate Identity
- Check device + inode to ensure same file

#### Prefer Descriptors
- Use file descriptors or `FILE*` instead of file names
- Avoids TOCTOU race conditions

#### Avoid TOCTOU
- Don't check file properties then open
- Open first, then check

#### Reusing Files
- Keep file open
- Use `lseek()` as needed

### Temporary Files & Permissions

#### mkstemp()
- Use `mkstemp()` for temporary files
- Creates with secure permissions

#### Restrictive umask
- Set restrictive `umask` before creating files

#### Avoid Shared Temp Directories
- When possible, use private temporary directories

#### Delete on Error
- Remove temporary files if operation fails

#### Explicit Modes
- Pass explicit modes to `open()`
- Prefer wrapper `OpenMode()` function

### Recommended Wrappers/Macros
- **dbcWrappers.h APIs**: Use wrapper functions
- **ARRAY_SIZE()**: Get array size
- **M4_INCR**: Macro increment
- **SystemCallWrapper()**: Wrapper for system calls
- **FREE()**: Safe free with NULL assignment

---

## 9. ERROR-HANDLING & ROBUSTNESS (MUST)

### Check All Return Values
- **Check all function return values**
- Handle all expected error codes
- Example:
  ```c
  int result = someFunction();
  if (result != SUCCESS) {
      // Handle error
      DEBUG1(caller():someFunction() failed with code %d, result);
      return ERROR;
  }
  ```

### Initialize Everything

#### File Descriptors
- Initialize to `-1`
- Example: `int fd = -1;`

#### Pointers
- Initialize to `NULL`
- Example: `char *ptr = NULL;`

### Prevent Out-of-Bounds (OOB)

#### Verify Buffer Sizes
- Always check bounds before access

#### Prefer Bounded Copies
- Use `strncpy()`, `snprintf()`, etc.

#### strncpy() NUL Behavior
- Be aware it may not NUL-terminate
- Use `Strncpy()` wrapper when size is known in context

### Resource Management

#### Release on All Exit Paths
- Use cleanup labels and goto if needed
- Example:
  ```c
  int processData(void) {
      char *buffer = NULL;
      int fd = -1;
      int result = ERROR;
      
      buffer = malloc(SIZE);
      if (buffer == NULL) {
          goto cleanup;
      }
      
      fd = open("file", O_RDONLY);
      if (fd < 0) {
          goto cleanup;
      }
      
      // Processing...
      result = SUCCESS;
      
  cleanup:
      if (buffer != NULL) {
          free(buffer);
      }
      if (fd >= 0) {
          close(fd);
      }
      return result;
  }
  ```

#### Static Arrays for Frequent Functions
- For frequent or periodic functions, prefer static arrays over frequent alloc/free
- Reduces overhead and potential memory leaks

---

## 10. FORMATTING (SHOULD)

### Artistic Style (astyle)
- Use Artistic Style via `make pretty` in LTIB
- Changes from `make pretty` **do not require review**

### Automatic Formatting
- Run before committing
- Ensures consistent code style across the project

---

## AGENT OUTPUT REQUIREMENTS (ALWAYS)

When generating code, agents must produce:

### 1. Conformant Code
- No banned patterns
- All rules above enforced

### 2. Proper Headers
- Module/function headers with descriptions
- Comments explain "why", not "how"

### 3. Safety Guards
- Input validation
- Bounds checks
- Return-value checks
- Secure I/O

### 4. Debug Macros
- Use `DEBUGn(fn_name():<msg>)`
- Never remove (gate with `ENABLE_DEBUG`)

### 5. Conditional Compilation
- Correct for Gas/Liquid/8-Path code
- Use `#if defined(...)` pattern

### 6. Self-Audit Checklist

After generating code, output **YES/NO** for each item:

- [ ] **Naming**: Types/vars/funcs/units/pointers/DB labels correct?
- [ ] **Functions**: Single return; boolean docs; module prefix for globals?
- [ ] **Headers**: Include guards; info hiding?
- [ ] **Style**: Lines ≤ 140; comment styles used?
- [ ] **system()**: Guarded with WIF*/WEXITSTATUS and logs?
- [ ] **Inputs/strings**: Sanitized; realpath() for paths?
- [ ] **Integers/pointers**: Bounds checks; overflow prevention?
- [ ] **Error handling/resources**: Returns checked; all resources freed?
- [ ] **Model guards**: Correct #if defined(...) usage?

**If ANY answer is NO**: Immediately provide revised, compliant code.

---

## AUTO-FIX RULES (WHEN NON-COMPLIANT)

When code violates guidelines, automatically apply these fixes:

### String Operations
- Replace `strcpy()` → `strncpy()` with explicit NUL termination
- Replace `sprintf()` → `snprintf()`
- Replace `gets()` → `fgets()`

### Bounds Checking
- Add range checks for array access
- Convert indices to `size_t`
- Split arithmetic to pre-check overflow

### Headers
- Insert include guards
- Move non-API details out of headers

### Conditional Compilation
- Wrap model-specific code with proper `#if defined` guards

### Debug Output
- Convert ad-hoc prints to `DEBUGn(fn_name():...)`

### system() Calls
- Add WIF*/WEXITSTATUS checks
- Add meaningful logging around `system()`

### Initialization
- Initialize uninitialized variables
- Set freed pointers to `NULL` (use `FREE()`)

---

## BANNED / DISCOURAGED PATTERNS

### Completely Banned
- ❌ `gets()` - use `fgets()` instead
- ❌ Unchecked `system()` - must check WIFEXITED, WEXITSTATUS, etc.
- ❌ User input as format string - never `sprintf(buffer, userInput)`
- ❌ Bitwise ops on signed types - only on unsigned
- ❌ Invalid shifts (negative or ≥ bit width)

### Discouraged (Avoid)
- ⚠️ Unbounded `strcpy()` - use `strncpy()` with explicit NUL termination
- ⚠️ Unbounded `sprintf()` - use `snprintf()`
- ⚠️ Mixing bitwise and arithmetic on same data
- ⚠️ Global variables - minimize usage
- ⚠️ Multiple returns in functions - prefer single return at end

---

## Quick Reference Examples

### Good Variable Names
```c
// Local variables (lowerCamelCase)
int userId;
size_t bufferSize;
char *pCurrentPath;

// Global/module variables (CamelCase)
CONFIG_STRUCT GlobalConfig;
STATUS_ENUM ModuleStatus;

// Physical units
float VolumeM3;
float FlowRateKgHr;
UINT32 TemperatureC;

// Booleans (positive)
BOOL isConnected;
BOOL hasValidData;
BOOL isPathGood;
```

### Good Function Names
```c
// Local functions (lowerCamelCase)
static int parseConfiguration(const char *pConfig);
static BOOL validateInput(const INPUT_STRUCT *pInput);

// Global functions (CamelCase with module prefix)
STATUS_ENUM BMEAS_Init(const CONFIG_STRUCT *pConfig);
void BMEAS_ProcessData(const DATA_STRUCT *pData);
BOOL USRMGMT_IsUserValid(UINT32 userId);
```

### Good Header Structure
```c
#if !defined(MODULE_NAME_H)
#define MODULE_NAME_H

// Public type definitions
typedef struct PUBLIC_DATA_STRUCT {
    UINT32 field1;
    UINT32 field2;
} PUBLIC_DATA_STRUCT;

// Public function declarations
STATUS_ENUM MODULE_Init(void);
void MODULE_Cleanup(void);

#endif // MODULE_NAME_H
```

### Good Error Handling
```c
STATUS_CODE_ENUM processData(const char *pInput, char *pOutput) {
    char *buffer = NULL;
    int fd = -1;
    STATUS_CODE_ENUM status = STATUS_ERROR;
    
    // Validate inputs
    if (pInput == NULL || pOutput == NULL) {
        DEBUG1(processData():Invalid NULL parameter);
        goto cleanup;
    }
    
    // Allocate resources
    buffer = malloc(BUFFER_SIZE);
    if (buffer == NULL) {
        DEBUG1(processData():Failed to allocate buffer);
        goto cleanup;
    }
    
    // Open file
    fd = open("datafile", O_RDONLY);
    if (fd < 0) {
        DEBUG1(processData():Failed to open file, errno=%d, errno);
        goto cleanup;
    }
    
    // Process data...
    status = STATUS_OK;
    
cleanup:
    if (buffer != NULL) {
        FREE(buffer);  // Also sets buffer = NULL
    }
    if (fd >= 0) {
        close(fd);
    }
    
    return status;
}
```

---

## Summary

This style guide ensures:
- **Consistency**: Uniform code style across the project
- **Safety**: Protection against common vulnerabilities
- **Maintainability**: Clear, understandable code
- **Reliability**: Proper error handling and resource management

All developers and code-generating agents must follow these guidelines without exception.
