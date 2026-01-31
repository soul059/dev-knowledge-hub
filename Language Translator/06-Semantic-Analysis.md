# Chapter 6: Semantic Analysis

## 6.1 Introduction to Semantic Analysis

**Semantic Analysis** is the phase that checks whether the syntactically correct program has meaningful semantics according to the language rules.

Problem: Parsing only checks structure. It cannot detect incorrect meaning like type mismatches or undeclared variables.

Solution: Semantic analysis adds meaning checks using the parse tree and the symbol table. It verifies types, declarations, and correct usage before code generation.

### What Syntax Analysis Cannot Catch

The parser only checks structure, not meaning:

```c
// Syntactically correct, but semantically wrong
int x = "hello";          // Type mismatch
y = x + 5;                // y not declared
int arr[10];
arr[20] = 5;              // Array bounds (may not be caught)
return x + func(x, x);    // func expects 1 argument, not 2
```

### Role of Semantic Analyzer

```
┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│   Parser     │───▶│  Semantic    │───▶│  Intermediate    │
│ (Parse Tree) │    │  Analyzer    │    │  Code Generator  │
└──────────────┘    └──────────────┘    └──────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Symbol Table │
                    └──────────────┘
```

### Tasks of Semantic Analysis

1. **Type Checking**: Ensure type compatibility in operations
2. **Scope Resolution**: Verify variables are declared before use
3. **Declaration Checking**: Ensure no duplicate declarations in same scope
4. **Function Calls**: Verify correct number and types of arguments
5. **Array Bounds**: Check array indices (when possible)
6. **Type Coercion**: Insert implicit type conversions

---

## 6.2 Syntax-Directed Definitions (SDD)

A **Syntax-Directed Definition** associates semantic rules with grammar productions to compute attribute values.

Problem: We need a systematic way to attach semantic meaning to syntax so the compiler can compute values and types.

Solution: SDD attaches attributes and rules to grammar productions so semantic values are computed alongside parsing.

### Components

1. **Attributes**: Values associated with grammar symbols
2. **Semantic Rules**: Computations that define attribute values

### Example SDD for Simple Calculator

**Grammar with Semantic Rules:**

| Production | Semantic Rule |
|------------|---------------|
| L → E \n | print(E.val) |
| E → E₁ + T | E.val = E₁.val + T.val |
| E → T | E.val = T.val |
| T → T₁ * F | T.val = T₁.val × F.val |
| T → F | T.val = F.val |
| F → ( E ) | F.val = E.val |
| F → digit | F.val = digit.lexval |

### Annotated Parse Tree

For input `3 * 5 + 4`:

```
                L (print 19)
                │
               E.val = 19
              / │ \
     E.val=15   +   T.val=4
         │          │
     T.val=15    F.val=4
      / │ \         │
T.val=3 * F.val=5  digit(4)
    │       │
 F.val=3  digit(5)
    │
 digit(3)
```

---

## 6.3 Types of Attributes

### Synthesized Attributes

**Definition**: An attribute of a node is **synthesized** if its value is computed from the attribute values of its **children**.

**Flow**: Bottom-up (from leaves to root)

**Example**:
```
E.val = E₁.val + T.val
```
The value of E is computed from values of its children E₁ and T.

Problem: Many semantic values are determined by the parts of an expression, but we need a clear direction of computation.

Solution: Synthesized attributes flow bottom-up from children to parent.

### Inherited Attributes

**Definition**: An attribute of a node is **inherited** if its value is computed from attributes of its **parent** or **siblings**.

**Flow**: Top-down or sideways

**Example**:
```
For: D → T L
     T → int | float
     L → L₁, id | id

Semantic rules:
     L.type = T.type    (L inherits type from T through D)
     L₁.type = L.type   (inherited down the L chain)

Problem: Some attributes depend on context (parent or siblings), not just children.

Solution: Inherited attributes allow passing information top-down or left-to-right across siblings.
```

### S-Attributed vs L-Attributed Definitions

| Type | Attributes | Evaluation Order |
|------|------------|------------------|
| **S-Attributed** | Only synthesized | Bottom-up (post-order) |
| **L-Attributed** | Synthesized + limited inherited | Left-to-right, depth-first |

**L-Attributed Condition**: For A → X₁X₂...Xₙ, inherited attribute of Xᵢ can depend only on:
- Inherited attributes of A
- Attributes of X₁, X₂, ..., Xᵢ₋₁ (left siblings)

---

## 6.4 Syntax-Directed Translation Schemes (SDT)

An **SDT** embeds semantic actions within production rules, specifying when actions execute.

Problem: Even with SDD, we still need to know when to run actions during parsing.

Solution: SDT places actions inside productions so the order of execution is explicit.

### Notation

Actions are enclosed in braces `{ }` within the production:

```
E → E₁ + T  { E.val = E₁.val + T.val; print('+'); }
```

### Postfix SDT

Actions at the **end** of productions (for S-attributed definitions):

```
E → E₁ + T    { E.val = E₁.val + T.val }
E → T         { E.val = T.val }
```

### SDT for Infix to Postfix Translation

```
E → T R
R → + T { print('+') } R | ε
T → id { print(id.name) }
```

For input `a + b + c`:
- Parse: E → T R → id R → a R → a + T R → a + b R → ...
- Output: `a b + c +`

---

## 6.5 Implementing SDT in Recursive Descent

### S-Attributed SDD Implementation

Actions after all children are processed:

```c
int E() {
    int val1 = T();      // Get T's value
    int val2 = E_prime(val1);  // Pass to E'
    return val2;
}

int E_prime(int in) {
    if (lookahead == '+') {
        match('+');
        int tval = T();
        int result = in + tval;
        return E_prime(result);  // Continue with accumulated value
    }
    return in;  // ε production
}

int T() {
    if (lookahead == '(') {
        match('(');
        int val = E();
        match(')');
        return val;
    } else {
        int val = number_value;
        match(NUMBER);
        return val;
    }
}
```

### L-Attributed SDD Implementation

Inherited attributes become function parameters:

```c
void L(Type inh_type) {  // inh_type is inherited
    if (lookahead == ID) {
        add_to_symbol_table(current_id, inh_type);
        match(ID);
    } else {
        L(inh_type);
        match(',');
        add_to_symbol_table(current_id, inh_type);
        match(ID);
    }
}
```

---

## 6.6 Type Systems

### What is a Type?

A **type** is a set of values together with operations that can be performed on them.

Problem: Without a formal type system, the compiler cannot decide which operations are valid.

Solution: A type system defines legal values and operations, enabling consistent type checking.

### Common Type Categories

| Category | Examples |
|----------|----------|
| **Primitive** | int, float, char, bool |
| **Composite** | arrays, records, structures |
| **User-defined** | classes, enumerations |
| **Pointer** | int*, char* |
| **Function** | int → int, (int, int) → bool |

### Type Expressions

Types can be represented as expressions:

```
Basic types: integer, float, char, boolean

Type constructors:
- array(I, T)     - array of T with index set I
- pointer(T)      - pointer to T  
- record(...)     - record/struct
- T₁ × T₂         - Cartesian product
- T₁ → T₂         - function from T₁ to T₂
```

**Example:**
```c
int arr[10];        // array(0..9, integer)
int *ptr;           // pointer(integer)
int func(int, int); // integer × integer → integer
```

---

## 6.7 Type Checking

### Static vs Dynamic Type Checking

| Aspect | Static | Dynamic |
|--------|--------|---------|
| When | Compile time | Runtime |
| Languages | C, Java, C++ | Python, JavaScript |
| Errors | Caught early | May cause runtime crashes |
| Overhead | None at runtime | Runtime checks needed |

Problem: Programs can perform invalid operations like adding a number to a string.

Solution: Type checking enforces rules and either rejects invalid programs (static) or checks at runtime (dynamic).

### Type Checking Rules

#### Assignment Compatibility

```
For: x = expr

Check: type(expr) can be assigned to type(x)

Rules:
- Same types: Always OK
- Widening: int → float OK
- Narrowing: float → int (may need explicit cast)
```

#### Arithmetic Operations

```
For: E₁ op E₂  (where op ∈ {+, -, *, /})

Type rules:
- int op int → int
- float op float → float
- int op float → float (coerce int to float)
- float op int → float (coerce int to float)
```

### Type Checking Implementation

```c
Type typecheck_binary(char op, Type t1, Type t2) {
    if (t1 == TYPE_ERROR || t2 == TYPE_ERROR)
        return TYPE_ERROR;
    
    if (op == '+' || op == '-' || op == '*' || op == '/') {
        if (t1 == TYPE_INT && t2 == TYPE_INT)
            return TYPE_INT;
        if ((t1 == TYPE_FLOAT || t1 == TYPE_INT) &&
            (t2 == TYPE_FLOAT || t2 == TYPE_INT))
            return TYPE_FLOAT;
        
        error("Type mismatch in arithmetic operation");
        return TYPE_ERROR;
    }
    
    if (op == '<' || op == '>' || op == EQ || op == NE) {
        if ((t1 == TYPE_INT || t1 == TYPE_FLOAT) &&
            (t2 == TYPE_INT || t2 == TYPE_FLOAT))
            return TYPE_BOOL;
        
        error("Type mismatch in comparison");
        return TYPE_ERROR;
    }
    
    return TYPE_ERROR;
}
```

---

## 6.8 Type Coercion and Conversion

### Implicit Coercion

The compiler automatically converts types when safe:

Problem: Mixed-type expressions are common, but operands must have compatible types.

Solution: Coercion inserts safe conversions (usually widening) so expressions can be evaluated correctly.

```
Widening conversions (safe):
char → int → long → float → double

Example:
    int a = 5;
    float b = 3.14;
    float c = a + b;  // a is implicitly converted to float
```

### Explicit Conversion (Casting)

Programmer explicitly requests conversion:

```c
float x = 3.7;
int y = (int)x;      // y = 3 (truncation)

int a = 65;
char c = (char)a;    // c = 'A'
```

### Type Coercion in Expressions

For expression `E₁ op E₂`:

```
if type(E₁) = integer and type(E₂) = integer:
    result type = integer
    no conversion needed

if type(E₁) = integer and type(E₂) = real:
    insert conversion node: inttoreal(E₁)
    result type = real

if type(E₁) = real and type(E₂) = integer:
    insert conversion node: inttoreal(E₂)
    result type = real

if type(E₁) = real and type(E₂) = real:
    result type = real
    no conversion needed
```

---

## 6.9 Symbol Table

### Purpose

The **Symbol Table** is a data structure that stores information about identifiers (variables, functions, types) used in the program.

Problem: The compiler must track identifiers across scopes with their types and locations.

Solution: The symbol table stores identifier metadata and provides lookup/insert operations during semantic analysis.

### Information Stored

| Attribute | Description |
|-----------|-------------|
| Name | The identifier string |
| Type | Data type (int, float, function, etc.) |
| Scope | Where the identifier is valid |
| Memory Location | Offset or address |
| Size | Memory size required |
| Value | For constants |
| Parameters | For functions |

### Symbol Table Entry Example

```
┌──────────────────────────────────────────────────┐
│ Name: "count"                                     │
│ Type: integer                                     │
│ Scope: local (function: main)                    │
│ Offset: 8                                         │
│ Size: 4 bytes                                     │
│ Line declared: 10                                 │
└──────────────────────────────────────────────────┘
```

### Operations

| Operation | Description |
|-----------|-------------|
| **Insert** | Add new identifier |
| **Lookup** | Find identifier information |
| **Delete** | Remove identifier (scope exit) |
| **Modify** | Update identifier attributes |

---

## 6.10 Scope Management

### Types of Scope

1. **Global Scope**: Visible throughout program
2. **Local Scope**: Visible within function/block
3. **Block Scope**: Visible within { }
4. **File Scope**: Visible within source file

Problem: Identifiers can be reused in different blocks, so the compiler must know which declaration is in effect.

Solution: Scope management ensures lookup always finds the nearest valid declaration and prevents duplicates in the same scope.

### Scope Example

```c
int x;                      // Global scope

void func() {
    int y;                  // Local to func
    if (condition) {
        int z;              // Block scope
        x = y + z;          // All visible
    }
    // z not visible here
}
// y not visible here
```

### Implementing Scope with Symbol Table

**Method 1: Single Table with Scope Field**
```
Each entry has a scope number
On scope entry: increment scope counter
On scope exit: remove entries with current scope, decrement counter
```

**Method 2: Stack of Tables (Most Common)**

```
┌─────────────────┐
│ Current Block   │ ← top (innermost scope)
├─────────────────┤
│ Enclosing Block │
├─────────────────┤
│     ...         │
├─────────────────┤
│ Global Scope    │ ← bottom
└─────────────────┘

Enter scope: Push new table
Exit scope: Pop table
Lookup: Search from top to bottom
Insert: Add to top table
```

### Symbol Table Implementation

```c
#define TABLE_SIZE 100

typedef struct Symbol {
    char *name;
    Type type;
    int offset;
    struct Symbol *next;  // For chaining
} Symbol;

typedef struct Scope {
    Symbol *table[TABLE_SIZE];
    struct Scope *parent;
} Scope;

Scope *current_scope = NULL;

// Hash function
unsigned hash(char *s) {
    unsigned h = 0;
    while (*s)
        h = h * 65599 + *s++;
    return h % TABLE_SIZE;
}

// Enter new scope
void enter_scope() {
    Scope *new_scope = malloc(sizeof(Scope));
    memset(new_scope->table, 0, sizeof(new_scope->table));
    new_scope->parent = current_scope;
    current_scope = new_scope;
}

// Exit scope
void exit_scope() {
    Scope *old = current_scope;
    current_scope = current_scope->parent;
    // Free old scope entries
    free(old);
}

// Insert symbol
void insert(char *name, Type type, int offset) {
    unsigned h = hash(name);
    Symbol *sym = malloc(sizeof(Symbol));
    sym->name = strdup(name);
    sym->type = type;
    sym->offset = offset;
    sym->next = current_scope->table[h];
    current_scope->table[h] = sym;
}

// Lookup symbol (search all scopes)
Symbol* lookup(char *name) {
    unsigned h = hash(name);
    for (Scope *s = current_scope; s != NULL; s = s->parent) {
        for (Symbol *sym = s->table[h]; sym != NULL; sym = sym->next) {
            if (strcmp(sym->name, name) == 0)
                return sym;
        }
    }
    return NULL;  // Not found
}
```

---

## 6.11 Semantic Analysis of Declarations

### Processing Declarations

```c
// Grammar rule: D → T id
void process_declaration(Type type, char *id) {
    // Check for duplicate in current scope
    if (lookup_current_scope(id) != NULL) {
        error("Duplicate declaration of '%s'", id);
        return;
    }
    
    // Calculate offset
    int offset = allocate_space(type);
    
    // Insert into symbol table
    insert(id, type, offset);
}

Problem: Declarations introduce names and types, and duplicates in the same scope are errors.

Solution: Check for duplicates in current scope, allocate storage, and insert into the symbol table.
```

### Processing Function Declarations

```c
// func_decl → type id ( params ) block
void process_function(Type return_type, char *name, ParamList params) {
    // Check for duplicate function
    if (lookup_global(name) != NULL) {
        error("Function '%s' already declared", name);
        return;
    }
    
    // Create function type
    Type func_type = make_function_type(return_type, params);
    
    // Insert function into global symbol table
    insert_global(name, func_type);
    
    // Enter new scope for function body
    enter_scope();
    
    // Add parameters to local scope
    for each param in params:
        insert(param.name, param.type, param.offset);
}
```

---

## 6.12 Semantic Analysis of Statements and Expressions

### Assignment Statement

```c
// stmt → id = expr
void check_assignment(char *id, Expr expr) {
    Symbol *sym = lookup(id);
    
    if (sym == NULL) {
        error("Undeclared variable '%s'", id);
        return;
    }
    
    if (!type_compatible(sym->type, expr.type)) {
        error("Type mismatch in assignment");
        return;
    }
    
    // May need to insert coercion
    if (needs_coercion(sym->type, expr.type)) {
        insert_coercion(expr, sym->type);
    }
}

Problem: Assignment must use a declared variable and compatible types.

Solution: Look up the variable, check type compatibility, and insert coercions if needed.
```

### If Statement

```c
// stmt → if ( expr ) stmt₁ else stmt₂
void check_if(Expr condition) {
    if (condition.type != TYPE_BOOL) {
        error("Condition must be boolean");
    }
}
```

### While Statement

```c
// stmt → while ( expr ) stmt
void check_while(Expr condition) {
    if (condition.type != TYPE_BOOL) {
        error("While condition must be boolean");
    }
    // Track that we're in a loop (for break/continue checking)
    loop_depth++;
}
```

### Function Call

```c
// expr → id ( args )
Type check_function_call(char *name, ArgList args) {
    Symbol *func = lookup(name);
    
    if (func == NULL) {
        error("Undefined function '%s'", name);
        return TYPE_ERROR;
    }
    
    if (!is_function(func->type)) {
        error("'%s' is not a function", name);
        return TYPE_ERROR;
    }
    
    ParamList params = get_params(func->type);
    
    // Check argument count
    if (length(args) != length(params)) {
        error("Wrong number of arguments");
        return TYPE_ERROR;
    }
    
    // Check argument types
    for (int i = 0; i < length(args); i++) {
        if (!type_compatible(args[i].type, params[i].type)) {
            error("Argument %d type mismatch", i+1);
        }
    }
    
    return get_return_type(func->type);
}
```

### Array Access

```c
// expr → id [ expr ]
Type check_array_access(char *name, Expr index) {
    Symbol *sym = lookup(name);
    
    if (sym == NULL) {
        error("Undeclared array '%s'", name);
        return TYPE_ERROR;
    }
    
    if (!is_array(sym->type)) {
        error("'%s' is not an array", name);
        return TYPE_ERROR;
    }
    
    if (index.type != TYPE_INT) {
        error("Array index must be integer");
        return TYPE_ERROR;
    }
    
    return get_element_type(sym->type);
}
```

---

## 6.13 Type Equivalence

### Structural Equivalence

Two types are equivalent if they have the **same structure**.

Problem: The compiler must decide when two types are considered the same.

Solution: Use structural equivalence (same shape) or name equivalence (same declared name) based on language rules.

```
typedef int Age;
typedef int Count;

// Age and Count are structurally equivalent (both are int)
```

### Name Equivalence

Two types are equivalent only if they have the **same name**.

```
typedef int Age;
typedef int Count;

// Age and Count are NOT name equivalent (different names)
```

### Implementing Structural Equivalence

```c
bool structurally_equivalent(Type t1, Type t2) {
    if (t1 == t2) return true;  // Same type object
    
    if (kind(t1) != kind(t2)) return false;
    
    switch (kind(t1)) {
        case BASIC:
            return basic_type(t1) == basic_type(t2);
            
        case ARRAY:
            return structurally_equivalent(element_type(t1), 
                                          element_type(t2)) &&
                   array_size(t1) == array_size(t2);
            
        case POINTER:
            return structurally_equivalent(pointed_type(t1),
                                          pointed_type(t2));
            
        case FUNCTION:
            return structurally_equivalent(return_type(t1),
                                          return_type(t2)) &&
                   param_types_equivalent(params(t1), params(t2));
            
        case RECORD:
            return fields_equivalent(fields(t1), fields(t2));
    }
    return false;
}
```

---

## 6.14 Attribute Grammar Example: Type Checking

Complete attribute grammar for expression type checking:

```
Grammar:
    E → E₁ + T
    E → T
    T → T₁ * F
    T → F
    F → ( E )
    F → id
    F → num

Semantic Rules:

E → E₁ + T
    E.type = if (E₁.type == int && T.type == int)
             then int
             else if (E₁.type == float || T.type == float)
             then float
             else error

E → T
    E.type = T.type

T → T₁ * F
    T.type = if (T₁.type == int && F.type == int)
             then int
             else if (T₁.type == float || F.type == float)
             then float
             else error

T → F
    T.type = F.type

F → ( E )
    F.type = E.type

F → id
    F.type = lookup(id.name).type

F → num
    F.type = int
```

---

## 6.15 Summary

### Key Concepts:

1. **Semantic Analysis** checks meaning beyond syntax
   - Type checking
   - Scope resolution
   - Declaration verification

2. **Syntax-Directed Definitions** attach semantic rules to grammar
   - **Synthesized attributes**: Computed from children
   - **Inherited attributes**: Computed from parent/siblings

3. **Type Systems** define valid operations
   - Type checking ensures operations are meaningful
   - Type coercion handles implicit conversions

4. **Symbol Table** stores identifier information
   - Operations: insert, lookup, delete
   - Scope management: stack of tables

5. **Type Equivalence**:
   - Structural: Same structure
   - Name: Same type name

Simple takeaway: Semantic analysis makes sure programs are not only well-formed, but also meaningful and type-correct before code generation.

---

## 🔍 Practice Questions

1. Design an SDD to compute the type of expressions with int and float operands.

2. Implement a symbol table with scope management for a simple language.

3. Write semantic rules for:
   - Variable declarations with initialization
   - Function declarations and calls

4. Explain the difference between S-attributed and L-attributed definitions.

5. What are the common semantic errors in programming languages? How are they detected?

---

*Next Chapter: [Intermediate Code Generation](07-Intermediate-Code-Generation.md)*
