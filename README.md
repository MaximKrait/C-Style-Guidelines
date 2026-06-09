## C++ Style Guidelines

Follow these formatting and architectural rules to keep the codebase consistent, high-performance, and clean.

---

### 1. Formatting & Whitespace

* **Indentation:** Use **4 spaces** for indentation. Do not use tabs.
* **Brace Placement:** Use **same-line braces** for functions, classes, conditionals, loops, and lambdas.
    ```cpp
    void handle_resize(int) { 
        resized = true; 
    }
    ```
* **No Spaces Around Parentheses:** Keep conditions and type casts compact.
    ```cpp
    if(c.current_mode == ENCODER_MODE)
    raw_buffer.push_back((uint8_t)(bit_cache >> (bits_count - 8)));
    ```
* **One-Line Blocks:** Short, single-line statements inside blocks or lambdas can be written on the same line to save vertical space.
    ```cpp
    {KEY_F(10), [](AppContext& c) { c.running = false; }},
    ```

---

### 2. Naming Conventions

* **snake_case Everything:** Use lower-case letters with underscores for variables, arguments, functions, structures, and enums.
    ```cpp
    int cursor_pos = 0;
    void handle_viewer_mouse(...);
    struct AppContext { ... };
    ```
* **SCREAMING_CASE for Constants & Enum Values:** Macros, global state variables, or specific enum values that act as global constants must be fully capitalized.
    ```cpp
    enum Mode { VIEWER_MODE, ENCODER_MODE };
    if (ch == GLP_META)
    ```
* **Keep Names Compact:** Avoid overly verbose names (`multipleLongWords`). Prefer short, clear variable and argument names (`ctx` or `c` for context, `ch` for character, `in`/`out` for streams).

---

### 3. Architecture & Data Structures

* **Avoid Excessive Getters/Setters:** Use raw `struct` fields or references (`&`) inside context structures to allow direct data manipulation. Encapsulation should only be added if strictly necessary.
    ```cpp
    struct AppContext {
        int& selected;
        Mode& current_mode;
        bool& decoder_active;
        std::wstring& input_text;
        int& cursor_pos;
        int total;
        bool& running;
    };
    ```
* **Data-Driven Logic Over Branches:** Prefer lookup tables (`std::map`, arrays of structs, or LUT arrays) instead of long `if/else` or `switch` chains for handling keys, commands, and translations.
    ```cpp
    std::map<wint_t, std::function<void(AppContext&)>> commands = { ... };
    
    struct KeyAction { wint_t key; std::function<void()> action; };
    KeyAction actions[] = { ... };
    ```

---

### 4. Memory & Performance

* **Pre-allocate Vector Capacity:** Always use `.reserve()` when the target size of a `std::vector` or `std::string` can be calculated beforehand. This eliminates unnecessary reallocations during bit-packing or file loading.
    ```cpp
    std::vector<uint8_t> raw_buffer;
    raw_buffer.reserve((glyph_sequence.size() * 3 + 7) / 8);
    ```
* **Compile-Time Initialization (LUT):** Use immediately-invoked lambdas (`[] { ... }()`) assigned to `static const` variables to generate heavy Look-Up Tables once at startup.
    ```cpp
    static const GlyphLut char_to_glyph_data = [] {
        GlyphLut lut = {};
        lut.table['T'] = GLP_T;
        return lut;
    }();
    ```
* **Pass by Reference:** Heavy objects (`std::string`, `std::wstring`, `AppContext`) must always be passed by reference (`const T&` or `T&`) to avoid deep copies.

---

### 5. Control Flow & Modern C++

* **No Braceless Loops/Conditionals:** Always wrap `if`, `else`, and `for` statements in braces, unless it is a highly specific single-line control phrase like `if (cond) return;`.
* **Lambda Captures:** Use explicit reference capture `[&]` for local, short-lived scopes (like UI action tables) where the lambda does not outlive the local variables.
* **C-Style API Integration:** When working with low-level libraries (`ncurses`, `zlib`), handle type conversions explicitly using `static_cast` or `reinterpret_cast` where needed, keeping the raw buffers wrapped inside modern containers (`std::vector::data()`).
