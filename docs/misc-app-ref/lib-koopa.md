# Libkoopa C 接口

`koopa.h` 头文件中定义了 Koopa IR 相关的 C 语言接口。通过这些接口，你可以解析文本形式的 Koopa IR，将其转换为内存中的数据结构（Raw Program），或者生成 LLVM IR 等。

使用该库时，需要在源文件中包含头文件：

```c
#include "koopa.h"
```

并在编译时链接 `libkoopa` 库（如果你使用模板 makefile/cmake 文件，我们已经帮你链接好了）。

所有接口的声明均在 `koopa.h` 中。此外，库中涉及的所有内存（如 `koopa_program_t` 和 `koopa_raw_program_builder_t`）都需要手动管理释放，具体请参考各函数的文档说明。


## 类型

## 结构体

## 枚举


## 函数

### koopa_parse_from_file

**描述**

从给定文件路径读取并解析文本形式的 Koopa IR 程序。如果未发生错误，将更新 `program` 指针。

**定义**

```c
koopa_error_code_t koopa_parse_from_file(const char *path,
                                         koopa_program_t *program);
```

**参数**

- `path`: 输入文件的路径字符串。
- `program`: 指向 `koopa_program_t` 的指针。解析成功后，该指针指向的内存将被更新为新创建的 Koopa IR 程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
koopa_error_code_t ret = koopa_parse_from_file("test.koopa", &program);
assert(ret == KOOPA_EC_SUCCESS);
// 使用 program ...
koopa_delete_program(program);
```

### koopa_parse_from_string

**描述**

从给定的字符串中解析文本形式的 Koopa IR 程序。如果未发生错误，将更新 `program` 指向的内容。

**定义**

```c
koopa_error_code_t koopa_parse_from_string(const char *str,
                                           koopa_program_t *program);
```

**参数**

- `str`: 包含 Koopa IR 程序代码的字符串。
- `program`: 指向 `koopa_program_t` 的指针。解析成功后，该指针指向的内存将被更新为新创建的 Koopa IR 程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
const char *str = "fun @main(): i32 { %entry: ret 0 }";
koopa_program_t program;
koopa_error_code_t ret = koopa_parse_from_string(str, &program);
assert(ret == KOOPA_EC_SUCCESS);
koopa_delete_program(program);
```

### koopa_parse_from_stdin

**描述**

从标准输入 (stdin) 读取并解析文本形式的 Koopa IR 程序。如果未发生错误，将更新 `program` 指向的内容。

**定义**

```c
koopa_error_code_t koopa_parse_from_stdin(koopa_program_t *program);
```

**参数**

- `program`: 指向 `koopa_program_t` 的指针。解析成功后，该指针指向的内存将被更新为新创建的 Koopa IR 程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
koopa_error_code_t ret = koopa_parse_from_stdin(&program);
// 此时程序会等待用户从标准输入输入 Koopa IR 代码
// 输入完毕后（需要发送 EOF，如 Linux 下 Ctrl+D），解析开始
assert(ret == KOOPA_EC_SUCCESS);
koopa_delete_program(program);
```

### koopa_parse_from_raw

**描述**

从给定的文件描述符 (UNIX/Linux) 或文件句柄 (Windows) 读取并解析文本形式的 Koopa IR 程序。如果未发生错误，将更新 `program` 指向的内容。

**定义**

```c
koopa_error_code_t koopa_parse_from_raw(koopa_raw_file_t file,
                                        koopa_program_t *program);
```

**参数**

- `file`: 已经打开的文件描述符 (`FILE*` 或 `int` 等，取决于具体平台定义 `koopa_raw_file_t`)。
- `program`: 指向 `koopa_program_t` 的指针。解析成功后，该指针指向的内存将被更新为新创建的 Koopa IR 程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_raw_file_t fd = open("hello.koopa", O_RDONLY);
if (fd != -1) {
  koopa_program_t program;
  koopa_error_code_t ret = koopa_parse_from_raw(fd, &program);
  assert(ret == KOOPA_EC_SUCCESS && "parsing koopa ir failure");

  close(fd);
  koopa_delete_program(program);
} else {
  assert(false && "Failed to open test file!");
}
```

### koopa_delete_program

**描述**

删除给定的 Koopa IR 程序对象，释放其占用的所有内存。
所有由 `koopa_parse_*` 系列函数返回的程序对象都必须在使用完毕后手动调用此函数删除，以避免内存泄漏。

**定义**

```c
void koopa_delete_program(koopa_program_t program);
```

**参数**

- `program`: 要删除的 Koopa IR 程序对象。

**示例**

```c
koopa_program_t program;
koopa_parse_from_string("...", &program);
// ... 处理 program ...
koopa_delete_program(program);
```

?> 为了避免忘记释放内存或者你觉得手动释放内存很麻烦，可以把 koopa 构造程序对象的过程封装进一个类使用 RAII。

### koopa_dump_to_file

**描述**

将 Koopa IR 程序以文本形式输出到指定文件。

路径可以是绝对路径，也可以是相对于当前工作目录的相对路径。

**定义**

```c
koopa_error_code_t koopa_dump_to_file(koopa_program_t program,
                                      const char *path);
```

**参数**

- `program`: 要输出的 Koopa IR 程序对象。
- `path`: 输出文件的路径。如果文件已存在，将被覆盖。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
koopa_parse_from_string("...", &program);
koopa_dump_to_file(program, "output.koopa");
koopa_delete_program(program);
```

### koopa_dump_to_string

**描述**

将 Koopa IR 程序以文本形式输出到给定的字符缓冲区，并在末尾添加 null 终止符。

如果 `buffer` 参数为 null，函数将计算生成字符串所需的长度（**不包含** null 终止符），并将结果存储在 `len` 指向的变量中。这通常用于确定缓冲区所需的最小大小。

`len` 同时代表了最多可以往 buffer 里写多少个字节（input）和实际上写了多少个字节（output）。

**定义**

```c
koopa_error_code_t koopa_dump_to_string(koopa_program_t program, char *buffer,
                                        size_t *len);
```

**参数**

- `program`: 要输出的 Koopa IR 程序对象。
- `buffer`: 指向字符缓冲区的指针，用于存储生成的字符串。可以为 null。
- `len`: 指向 `size_t` 变量的指针。
  - 如果 `buffer` 为 null，函数将写入所需的字符串长度。
  - 如果 `buffer` 不为 null，函数将写入实际写入缓冲区的字符数。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
// ... 初始化 program ...

size_t len;
auto ret = koopa_dump_to_string(program, nullptr, &len);
assert(ret == KOOPA_EC_SUCCESS && "dumping koopa ir failure");

fmt::print("Koopa IR length: {}\n", len);

len = len + 1;
std::string buffer(len, '\0');
ret = koopa_dump_to_string(program, buffer.data(), &len);
/*
len 同时代表了最多可以往 buffer 里写多少个字节（input）和实际上写了多少个字节（output）。
所以如果这里这么写：
去除掉 len = len + 1
std::string buffer(len + 1, '\0');
ret = koopa_dump_to_string(program, buffer.data(), &len);
ret 会是 KOOPA_EC_INSUFFICIENT_BUFFER_LENGTH
*/
assert(ret == KOOPA_EC_SUCCESS && "dumping koopa ir failure");
fmt::print("{}", buffer);

koopa_delete_program(program);
```

### koopa_dump_to_stdout

**描述**

将 Koopa IR 程序以文本形式输出到标准输出 (stdout)。

**定义**

```c
koopa_error_code_t koopa_dump_to_stdout(koopa_program_t program);
```

**参数**

- `program`: 要输出的 Koopa IR 程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
// ... 初始化 program ...
koopa_dump_to_stdout(program);
koopa_delete_program(program);
```

### koopa_dump_to_raw

**描述**

将 Koopa IR 程序以文本形式输出到给定的文件描述符 (UNIX/Linux) 或文件句柄 (Windows)。

**定义**

```c
koopa_error_code_t koopa_dump_to_raw(koopa_program_t program,
                                     koopa_raw_file_t file);
```

**参数**

- `program`: 要输出的 Koopa IR 程序对象。
- `file`: 已经打开并具备写权限的文件描述符。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
koopa_parse_from_file("hello.koopa", &program);

koopa_raw_file_t fd = open("output.koopa", O_WRONLY | O_CREAT | O_TRUNC, 0644);
assert(fd >= 0 && "Failed to open output file");

auto ret = koopa_dump_to_raw(program, fd);
assert(ret == KOOPA_EC_SUCCESS && "Failed to dump program to file");

close(fd);
koopa_delete_program(program);
```

### koopa_dump_llvm_to_file

**描述**

将 Koopa IR 程序转换为 LLVM IR，并将结果以文本形式输出到指定文件。

**定义**

```c
koopa_error_code_t koopa_dump_llvm_to_file(koopa_program_t program,
                                           const char *path);
```

**参数**

- `program`: 要转换的 Koopa IR 程序对象。
- `path`: 输出文件的路径。如果文件已存在，将被覆盖。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
// ... 初始化 program ...
koopa_dump_llvm_to_file(program, "output.ll");
koopa_delete_program(program);
```

### koopa_dump_llvm_to_string

**描述**

将 Koopa IR 程序转换为 LLVM IR，并将结果以文本形式输出到给定的字符缓冲区，并在末尾添加 null 终止符。

如果 `buffer` 参数为 null，则计算所需的缓冲区长度（**不包含** null 终止符），并存储在 `len` 中。

`len` 同时代表了最多可以往 buffer 里写多少个字节（input）和实际上写了多少个字节（output）。

**定义**

```c
koopa_error_code_t koopa_dump_llvm_to_string(koopa_program_t program,
                                             char *buffer, size_t *len);
```

**参数**

- `program`: 要转换的 Koopa IR 程序对象。
- `buffer`: 指向字符缓冲区的指针，用于存储生成的 LLVM IR 字符串。可以为 null。
- `len`: 指向 `size_t` 变量的指针，用于存储长度信息。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
// ... 初始化 program ...

size_t len;
auto ret = koopa_dump_llvm_to_string(program, nullptr, &len);
assert(ret == KOOPA_EC_SUCCESS && "dumping koopa ir failure");

fmt::print("Koopa IR length: {}\n", len);

len = len + 1;
std::string buffer(len, '\0');
ret = koopa_dump_llvm_to_string(program, buffer.data(), &len);
/*
len 同时代表了最多可以往 buffer
里写多少个字节（input）和实际上写了多少个字节（output）。 所以如果这里这么写：
去除掉 len = len + 1
std::string buffer(len + 1, '\0');
ret = koopa_dump_llvm_to_string(program, buffer.data(), &len);
ret 会是 KOOPA_EC_INSUFFICIENT_BUFFER_LENGTH
*/
assert(ret == KOOPA_EC_SUCCESS && "dumping koopa ir failure");
fmt::print("{}", buffer);

koopa_delete_program(program);
```

### koopa_dump_llvm_to_stdout

**描述**

将 Koopa IR 程序转换为 LLVM IR，并将结果以文本形式输出到标准输出 (stdout)。

**定义**

```c
koopa_error_code_t koopa_dump_llvm_to_stdout(koopa_program_t program);
```

**参数**

- `program`: 要转换的 Koopa IR 程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
// ... 初始化 program ...
koopa_dump_llvm_to_stdout(program);
koopa_delete_program(program);
```

### koopa_dump_llvm_to_raw

**描述**

将 Koopa IR 程序转换为 LLVM IR，并将结果以文本形式输出到给定的文件描述符 (UNIX/Linux) 或文件句柄 (Windows)。

**定义**

```c
koopa_error_code_t koopa_dump_llvm_to_raw(koopa_program_t program,
                                          koopa_raw_file_t file);
```

**参数**

- `program`: 要转换的 Koopa IR 程序对象。
- `file`: 已经打开并具备写权限的文件描述符。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program;
koopa_parse_from_file("hello.koopa", &program);

koopa_raw_file_t fd =
    open("output.ll", O_WRONLY | O_CREAT | O_TRUNC, 0644);
assert(fd >= 0 && "Failed to open output file");

auto ret = koopa_dump_llvm_to_raw(program, fd);
assert(ret == KOOPA_EC_SUCCESS && "Failed to dump program to file");

close(fd);
koopa_delete_program(program);
```

### koopa_new_raw_program_builder

**描述**

创建一个新的原始程序构建器 (`koopa_raw_program_builder_t`)。
该构建器用于将高级的 `koopa_program_t` 对象转换为更易于遍历和分析的 `koopa_raw_program_t` 结构。

**定义**

```c
koopa_raw_program_builder_t koopa_new_raw_program_builder();
```

**返回值**

- 返回新创建的构建器对象的指针。

**示例**

```c
koopa_raw_program_builder_t builder = koopa_new_raw_program_builder();
// ... 使用 builder ...
koopa_delete_raw_program_builder(builder);
```

### koopa_delete_raw_program_builder

**描述**

删除给定的原始程序构建器，并释放其占用的内存。必须在构建器不再使用后手动调用，以避免内存泄漏。

**定义**

```c
void koopa_delete_raw_program_builder(koopa_raw_program_builder_t builder);
```

**参数**

- `builder`: 要删除的构建器对象。

**示例**

```c
// 见 koopa_new_raw_program_builder 示例
```

?> 为了避免忘记释放内存或者你觉得手动释放内存很麻烦，可以把 koopa 构造程序对象的过程封装进一个类使用 RAII。

### koopa_build_raw_program

**描述**

使用给定的构建器，将 `koopa_program_t` 对象转换为 `koopa_raw_program_t` 结构。

`koopa_raw_program_t` 是 Koopa IR 的内存表示形式，包含了程序的完整结构信息（函数、基本块、指令等），便于遍历和分析。
生成的原始程序 (`koopa_raw_program_t`) 的生命周期依附于构建器 (`builder`)。只要构建器未被删除，原始程序就有效。

**定义**

```c
koopa_raw_program_t koopa_build_raw_program(koopa_raw_program_builder_t builder,
                                            koopa_program_t program);
```

**参数**

- `builder`: 已经创建好的原始程序构建器。
- `program`: 要转换的 Koopa IR 程序对象。

**返回值**

- 返回构建好的 `koopa_raw_program_t` 结构。

**示例**

```c
koopa_program_t program;
// ... 初始化 program ...

koopa_raw_program_builder_t builder = koopa_new_raw_program_builder();
koopa_raw_program_t raw = koopa_build_raw_program(builder, program);

// ... 遍历 raw program，进行分析或优化 ...

koopa_delete_raw_program_builder(builder);
koopa_delete_program(program);
```

### koopa_generate_raw_to_koopa

**描述**

将给定的原始程序 (`koopa_raw_program_t`) 逆向转换回 Koopa IR 程序对象 (`koopa_program_t`)。

`libkoopa` 的文本输出函数（如 `koopa_dump_to_string`）仅接受 `koopa_program_t`。如果你手动构建或修改了 `raw` 结构，并希望将其导出为 Koopa IR 文本形式，则可以使用此函数将其转换为 `program` 对象。

**定义**

```c
koopa_error_code_t koopa_generate_raw_to_koopa(const koopa_raw_program_t *raw,
                                               koopa_program_t *program);
```

**参数**

- `raw`: 指向原始程序结构的指针。
- `program`: 指向 `koopa_program_t` 的指针，用于存储生成的新程序对象。

**返回值**

- 返回 `KOOPA_EC_SUCCESS` 表示成功，否则返回相应的错误码。

**示例**

```c
koopa_program_t program1, program2;
// ... 初始化 program1 ...

koopa_raw_program_builder_t builder = koopa_new_raw_program_builder();
koopa_raw_program_t raw = koopa_build_raw_program(builder, program1);
// ... 修改 raw ...

// 将 raw program 转换回 program2
koopa_generate_raw_to_koopa(&raw, &program2);

// ... 使用 program2 ...

koopa_delete_program(program2);
koopa_delete_raw_program_builder(builder);
koopa_delete_program(program1);
```
