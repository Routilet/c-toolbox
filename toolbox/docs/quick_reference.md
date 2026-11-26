# C Toolbox 速查 · v@VERSION@

C99 · 设计者 Annouimo · 维护组织 Routilet

[开始做题](#开始做题) · [标量 IO](#标量-io) · [文本与数列](#文本与数列) · [按题目选工具](#按题目选工具) · [共同约定](#共同约定)

## 开始做题

1. 运行入口 Python 脚本，填写题目目录并选择程序框架。目录留空会在入口旁导出手册。
2. 默认只需在 `main.c` 中引入 `c_toolbox.h`。继续按下文查找内置 API。手动编译用 `cc -std=c99 -O2 main.c -o main -lm`。
3. 运行题目目录的 `prepare.py`，按 `main.c` 中引用的名字组装依赖、检查编译并复制。结果在 `.toolbox/submit.c`。可用 `--no-copy` 只检查和保存，`--run` 运行，`--cc` 指定编译器。

默认组装模式面向单个 `main.c`，仅支持工具箱和标准库的 `#include`。需要自定义容器、其他预处理指令或自有头文件时，用入口脚本的 `--modular` 创建目录，再按下文的 `ctb/` 路径引入头文件。已有目录不覆盖，每题保留创建时的版本。

## 标量 IO

| 任务 | 接口 |
| --- | --- |
| 标量 | read_int / read_ll / read_uint / read_ull / read_double / read_bool，无参数、返回值 |
| 检查读取 | try_read_int() 等，返回 status/value；status 与 IO_OK、IO_EOF、IO_FORMAT、IO_RANGE、IO_STREAM_ERROR 比较 |
| 字符 | read_char() 跳过空白；read_char_raw() 保留空白 |
| 输出 | print_int(x,end)、print_ll(x,end)、print_text(text,end)、print_char(ch,end)；例如 print_int(x,"\n") |
| 小数与格式 | print_double(x,digits,end)、print_fixed(x,digits,end)；digits=PRINT_AUTO 使用有效数字，固定小数自动消除负零。print_format(format,end,...) |
| 输入控制 | io_skip_line() 丢弃行尾；io_skip_whitespace() 跳过空白；io_flush() 刷新 stdout（交互题每次输出查询后调用） |
| 文件 | IOReader r=io_reader(file)；io_try_read_int(&r)、io_try_read_line_raw(&r) 等 |

普通数值读取按完整单词解析，不把 12abc 当成 12；失败报告错误并退出。检查接口不退出，失败时 value 仅作占位，必须先检查状态；IO_OK=0，不能当布尔值使用。同一流不混用 scanf 等其他读取器。

## 文本与数列

### String

例如 `String s = read_line(); print_string(&s, "\n"); string_free(&s);`。

| 任务 | 接口 |
| --- | --- |
| 读词或行 | `read_word()`、`read_line()`、`read_line_raw()`、`read_rest()`、`read_next_line()`、`read_next_line_raw()`。对应的 `try_read_*` 返回状态和值 |
| 数值解析 | `string_parse_int/ll/double` 返回 `ok/value` |
| 创建、释放 | string_new / string_from(text) / string_clone(&s) / string_repeat(&s,count) / string_repeat_char(ch,count)；string_free(&s) |
| 查询 | string_size/empty/compare/equal；string_cstr(&s) 返回借用的 const char*；string_at(&s,i) |
| 修改 | string_assign/append/insert/erase/set/push；append_slice/append_repeat；pad_left/pad_right；trim/lower/upper/reverse、`string_is_palindrome` 查询回文 |
| 切片 | string_slice(&s,pos,count)；string_slice_step(&s,first,last,step)，正步长左闭右开 |
| 字符查询 | `string_find_char(&s, ch, start)`、`string_contains_char(&s, ch)`、`string_count_char(&s, ch)` |
| 字符修改 | `string_replace_char(&s, from, to)`、`string_remove_char(&s, ch)` 返回匹配或删除的数量，原地处理所有匹配。`string_pop_char(&s)` 返回末字符，空串报错。`string_try_pop_char(&s)` 返回 ok/value |
| 字符排序和去重 | `string_sort_chars(&s)` 按无符号字节升序，如 banana → aaabnn。`string_unique_chars(&s)` 保留首次出现顺序，如 banana → ban，返回删除数量。两者都原地修改 |
| 子串查找 | string_find(&s,needle,start)，未找到 SIZE_MAX；contains/count/count_overlapping |
| 替换 | string_replace_all；remove_first/remove_all/remove_repeated |
| 格式化 | string_format(format,...)；string_append_format(&s,format,...) |
| 连续字符 | string_longest_char_run(&s,ch) 返回 begin/end，同长取最早 |

line 默认跳过空白行、修剪首尾，保留内部空格；raw 保留空行与首尾空白，移除 LF/CRLF/CR。数字后 raw 读取当前分隔符起的行尾；next_line_raw 跨过数字所在行再读下一完整行，遇到额外非空白数据返回 IO_EXTRA_DATA。read_rest 读取当前行剩余内容。 文本按字节索引，不支持内嵌 NUL。字符查找从 start 开始，未找到返回 SIZE_MAX。用 `string_find_s` 可直接查找另一个 String。

### StringList

例如 `StringList a = read_words(n); string_list_sort(&a); print_strings(&a, " ", "\n"); string_list_free(&a);`。

| 任务 | 接口 |
| --- | --- |
| 列表 | string_list_new/push/push_text/push_take/get/at/set/insert/erase/clear/clone/slice/swap/free |
| 列表弹出 | string_list_pop(&list) 返回拥有资源的 String；string_list_try_pop 返回 ok/value |
| 列表查找 | `string_list_find_text(&list, "abc", start)` 返回下标或 SIZE_MAX。另有 `string_list_contains_text`、`string_list_count_text`。`string_list_find` 接收 String 指针，从头开始 |
| 列表排序 | string_list_sort；sort_by（稳定）；argsort/argsort_by（返回原下标）；swap；reverse；unique_sorted |
| 分割、连接 | string_split(&s,delimiter) 保留空字段；string_split_whitespace；string_join(&list,separator) |

`string_remove_patterns` 会反复删除列表中的模式，直到文本稳定。列表插入默认复制字符串，`push_take` 转移所有权并清空源对象。`unique_sorted` 只合并相邻的相等项。`get` 是只读借用，`at` 是可写借用。修改列表可能使借用失效。

### Vector

例如 `IntVector v = read_ints(n); IntVector_sort_asc(&v); print_ints(&v, " ", "\n"); IntVector_free(&v);`。其他两种用 `read_lls`、`read_doubles` 和对应的 `print_lls`、`print_doubles`。double 输出在分隔符前加精度参数。

以下 _ 表示对应类型前缀，如 IntVector_push。内置 int、long long、double 向量具备稳定排序、统计和回调操作；整数 sum 返回宽类型并检查溢出。

| 任务 | 接口 |
| --- | --- |
| 创建 | _new()、_filled(n,value)、_from(items,count)、_values(1,2,3)；整数 _range(start,stop,step) 不含 stop |
| 修改 | _push、_insert、_erase、_set（越界报错）、_try_set（越界返回 false）、_resize、_fill、_reverse、_swap、_clear、_free |
| 取值 | _get(&v,i)、_at、_front、_back、_pop；检查版 _try_get / _try_pop 返回 ok/value。_in_bounds 可判断负下标 |
| 复制 | _clone、_slice(&v,pos,count)、_slice_step(&v,first,last,step)、_extend_vector，返回新对象的需释放 |
| 排序 | 数值 _sort_asc/desc；_sort_by 使用类型明确的比较器、稳定排序；_sort 使用 qsort 形比较器；所有内置向量提供 _swap |
| 按值查询 | 六种类型均有 `_find(&v, value)`、`_find_from(&v, value, start)`、`_contains`、`_count`。数值向量的 `_lower_bound/upper_bound` 要求升序，另有 `_unique_sorted` |
| 连续段 | `_longest_run(&v, value)` 返回左闭右开 begin/end，同长取最早，未命中为 0/0 |
| 单项累加 | `IntVector_add_at(&v, i, delta)` 原地修改。long long 和 double 同样支持 |
| 统计 | _sum/mean/variance/stddev/median/min/max/argmin/argmax |
| 函数操作 | _map/filter/reduce、_map_ctx/_filter_ctx/_reduce_ctx（回调带 void * 上下文）、any/all/count_if/find_if/map_inplace |

find 未找到返回 SIZE_MAX。double 的按值查找、计数和连续段使用精确相等。slice 是起点＋数量；slice_step 使用正步长和左闭右开区间，不支持负下标。空容器 sum=0，其余统计要求非空；variance 为总体方差；median 不修改输入；整数 sum 检查每步 long long 溢出。容器元素浅拷贝，拥有字符串的序列优先 StringList。

`CharVector` 可保存内嵌 NUL。`read_chars(n)` 读取 n 个非空白字符。`read_bools(n)` 读取 n 个值为 0 或 1 的整数，`print_bools(&v, sep, end)` 输出 0/1。

## 按题目选工具

### 矩阵

内置数值矩阵有 `IntMatrix`、`LongLongMatrix` 和 `DoubleMatrix`。字符网格用 `CharMatrix`，访问标记用 `BoolMatrix`。

下表的 `_` 前加类型名，例如 `IntMatrix_get(&m, r, c)`。行列从 0 开始，矩形参数统一为左上角行列、高度、宽度。

| 任务 | 接口 |
| --- | --- |
| 创建与释放 | `_new(rows, cols)`、`_filled(rows, cols, value)`、`_empty(&m)`、`_free(&m)`。用 `_rows`、`_cols`、`_size` 查询行数、列数和元素总数 |
| 访问单格 | `_get(&m, r, c)`、`_set(&m, r, c, value)`。用 `_in_bounds` 判断是否在界内，或用 `_try_get` 取 `ok/value`。`_in_bounds`、`_try_get` 和 `_try_set` 都将负坐标判为越界。`_set` 越界报错，`_try_set` 返回 false |
| 填充与画框 | `_fill`、`_fill_row`、`_fill_col`、`_fill_rect`、`_draw_rect`。带 `rect` 的接口在区域参数后接填充值 |
| 复制与变换 | `_clone`、`_swap`、`_transpose`、`_slice(&m, r, c, h, w)`。复制、转置和切片返回独立矩阵。`_resize(&m, rows, cols, value)` 保留左上角重叠区域，新格子用 value 填充 |
| 数值运算 | `_multiply(&a, &b)`、`_sum_rect(&m, r, c, h, w)`、`_sum_border(&m, r, c, h, w)`。边框的每格只算一次，含单行单列。整数求和返回 `long long`，整数求和及乘法检查溢出 |
| 数值读写 | `read_int_matrix(rows, cols)`、`print_int_matrix(&m, elem_sep, row_sep, end)`。可换成 `ll` 或 `double`，double 输出在 `elem_sep` 前加小数位数 |
| 字符网格读写 | `read_char_matrix(rows, cols)` 保留空格并检查行宽，接在数字等单词之后时从下一行读起。`print_char_matrix(&m, "", "\n", "\n")` 按行输出 |
| 单格累加 | `IntMatrix_add_at(&m, row, col, delta)` 原地修改。long long 和 double 同样支持 |
| 布尔矩阵读写 | `read_bool_matrix(rows, cols)` 按行读值为 0 或 1 的整数。`print_bool_matrix(&m, elem_sep, row_sep, end)`，格子输出为 0 或 1 |

行求和可用 `IntMatrix_sum_rect(&m, row, 0, 1, IntMatrix_cols(&m))`。列求和把宽度设为 1。

矩阵打印只在相邻元素间插入 `elem_sep`，只在相邻行间插入 `row_sep`，整个矩阵结束后追加一次 `end`。

### 序列与区间

| 任务 | 头文件 | 接口 |
| --- | --- | --- |
| 相等段、最大子段、相邻差分 | `ctb/sequence/algorithms.h` | `IntVector_longest_equal_run`、`IntVector_max_subarray`、`IntVector_difference` |
| 下一排列 | `ctb/vector/int.h` | `IntVector_next_permutation(&v)`，到最后一个排列后复位并返回 false |
| 稳定排序下标、离散化 | `ctb/sequence/algorithms.h` | 各数值 Vector 的 `_argsort` 和 `IntVector_discretize` |
| 静态区间求和 | `ctb/sequence/prefix_sum.h` | `prefix_sum_from(&v)` 创建快照，`prefix_sum_query(&p,begin,end)` 求和，`prefix_sum_free(&p)` 释放 |
| 批量区间加法 | `ctb/sequence/range_difference.h` | `range_difference_from_ints`、`range_difference_add(&r,first,last,delta)`、`range_difference_materialize`、`range_difference_free` |
| 枚举组合 | `ctb/sequence/combination.h` | `combination_new(n,k)`，每次 `combination_next(&c)` 成功后读取 `c.indices`，最后 `combination_free(&c)` |

区间用左闭右开。`argsort` 返回零基原下标，类型为 `IndexVector`（`ctb/vector/index.h`），用 `IndexVector_free` 释放。离散化结果含去重后的 `values` 和原顺序的 `ranks`，用 `int_discretization_free` 释放。

### 数值与进制

| 任务 | 头文件 | 接口与约定 |
| --- | --- | --- |
| 位 | `ctb/math/bits.h` | bit_get/set/clear/toggle(value,bit)，最低位 0；bit_count；bit_lowest_set_index / bit_highest_set_index，零返回 exists=false |
| GCD/LCM | `ctb/math/arithmetic.h` | gcd/lcm(a,b)，输入 long long，返回 unsigned long long；负数取绝对值，gcd(0,0)=0 |
| 阶乘、幂 | `ctb/math/arithmetic.h` | factorial(unsigned n) 返回 unsigned long long；integer_power(base,unsigned exponent) 返回 long long，0^0=1；modular_power(base,exponent,modulus) |
| 检查计算 | `ctb/math/arithmetic.h` | try_factorial/try_lcm/try_integer_power 返回 ok/value；modular_inverse(value,modulus) 在模数 2..LLONG_MAX 时返回 exists/value；普通版溢出退出 |
| 舍入 | `ctb/math/arithmetic.h` | round_digits(x,digits)，中点远离零，负 digits 表示十/百位；受浮点表示精度影响 |
| 浮点比较 | `ctb/math/arithmetic.h` | nearly_equal(a,b,absolute_tolerance,relative_tolerance) |
| 多项式 | `ctb/math/arithmetic.h` | polynomial_value(coefficients,count,x)，系数按次数升序 |
| 进制 | `ctb/math/number.h` | integer_to_base(value,base) 返回 String；integer_from_base(&s,base)，检查版 try_integer_from_base，base 2～36 |
| 数位 | `ctb/math/number.h` | integer_digit_count/sum(value,base)、integer_digit_at(value,base,position)、integer_reverse_digits；负数按绝对值处理，反转溢出可检查 |
| 素数 | `ctb/math/sieve.h` | is_prime(n) 试除；prime_sieve_new(limit)/contains/free；查询不能超出筛范围 |

**大整数**用 `ctb/math/bigint.h`。BigInt：bigint_new/from("-123")/from_ll/clone/compare/negate/free；try_bigint_from 返回 ok/value。add/subtract/multiply/divide/modulo 返回新对象；divmod 返回 quotient/remainder，两者都释放；multiply_small、divide_small（返回商余数）；factorial。read_bigint/try_read_bigint、print_bigint、bigint_to_string。

解析仅接受可选正负号和至少一个十进制数字。除法向零截断，余数与被除数同号，满足 a=q*b+r；除零退出。所有新算术结果独立拥有内存。采用十进制学校算法，适合普通习题，非百万位高性能库。

**复数**用 `ctb/math/complex.h`。Complex 是 C99 double complex：complex_new(real,imaginary)、real/imaginary/magnitude、add/subtract/multiply/divide；read_complex 读实虚两数，print_complex(z,digits,end)。值类型无须释放，除零退出。

### 搜索与日期

`ctb/search.h` 提供整数答案二分与数值求根。

| 任务 | 接口与约定 |
| --- | --- |
| 答案二分 | integer_first_true(begin,end,predicate)，闭区间、假到真；返回 ok/value |
| 二分求根 | bisect_root(function,begin,end,tolerance,iterations)，返回 ok/value；连续函数且端点异号，不保证检测不连续点 |

`ctb/date.h` 提供公历日期和时刻。Date 为公历 1～9999 年值类型：date_new/try_date/valid/is_leap/compare；date_difference(a,b) 为从 a 到 b 的天数；date_add_days/try_date_add_days。read_date 读三个空白分隔整数；print_date(end) 输出 YYYY-MM-DD。TimeOfDay 与 time_from_seconds/time_to_seconds 限一天内；Duration 支持不限 24 小时的非负时长，time_difference(a,b) 为同一天内有符号秒差；read_time 读三个整数，print_time(end) 输出 HH:MM:SS，无时区、夏令时。

## 共同约定

- 下标从 0 开始。`slice(pos, count)` 第二个区域参数是长度，`slice_step(begin, end, step)` 的 end 不包含在内，矩形用 `row, col, height, width`。
- Vector 和 Matrix 的 `set` 越界会报错退出，`try_set` 返回 false。`string_set` 在越界或尝试写入 NUL 时返回 false。
- `add_at` 在越界、整数溢出或 double 结果非有限时报告错误。
- 持有内存的对象用各自的 `free` 函数释放。赋值只复制结构体，独立副本用 `clone`。借用的指针可能在原对象修改后失效。
- 普通读取遇错退出。需要处理 EOF 或非法输入时，用 `try_read_*` 并检查 `status`。检查版容器操作先检查 `ok`，再使用 `value`。

## 分模块头文件

| API | `--modular` 模式的头文件 |
| --- | --- |
| 标量 IO | `ctb/io.h` |
| String、StringList | `ctb/string.h`、`ctb/string_list.h` |
| Vector | `ctb/vector/` 下的 `int.h`、`long_long.h`、`double.h`、`char.h`、`bool.h`、`index.h` |
| Matrix | `ctb/matrix/` 下的 `int.h`、`long_long.h`、`double.h`、`char.h`、`bool.h` |
| 自定义容器 | `ctb/vector/custom.h` 的 `CTB_VECTOR_DEFINE(Name, Type)`，或 `ctb/matrix/custom.h` 的 `CTB_MATRIX_DEFINE(Name, Type)`。先定义元素类型，元素按值浅拷贝 |
