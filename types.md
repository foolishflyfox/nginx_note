# Nginx 类型

## 基本类型

- `ngx_int_t`: 为 `intptr_t` 类型，与地址长度相同的整数，在 `<stdint.h>` 中。
- `ngx_flag_t`: 为 `intptr_t` 类型。
- `ngx_uint_t`: 为 `uintptr_t` 类型，与地址长度相同的无符号整数。

- `ngx_err_t`: 为 `int` 类型，与标准库中的 `errno` 类型相同。