# Scilla进阶

## Scilla合约架构

Scilla合约的一般结构如下代码所示。开头是引用纯数学函数的`library`的声明，例如计算两个比特数据的`AND` 布尔值，或计算某个自然数的阶乘。在库声明代码块后面，是使用`contract`声明的合约内容。合约有三个部分，第一部分声明合约的不可变参数，第二部分声明可变字段，第三部分为所有`transition`定义。

```
(* Scilla contract structure *)


(***************************************************)
(*               Associated library                *)
(***************************************************)

library MyContractLib


(* Library code block follows *)



(***************************************************)
(*             Contract definition                 *)
(***************************************************)

contract MyContract

(* Immutable fields declaration *)

(vname_1 : vtype_1,
 vname_2 : vtype_2)

(* Mutable fields declaration *)

field vname_1 : vtype_1 = init_val_1
field vname_2 : vtype_2 = init_val_2

(* Transitions *)


(* Transition signature *)
transition firstTransition (param_1 : type_1, param_2 : type_2)
  (* Transition body *)

 end

transition secondTransition (param_1: type_1)
  (* Transition body *)

end
```

### 不可变变量

不可变变量，或称为合约参数，在创建合约时就定义了值，并且不能被修改。合约中的不可变变量组需在合约开头的合约名称定义之后声明。

不可变变量声明格式如下：

```
(vname1 : type1,
 vname2 : type2,
  ...  )
```

每个声明都包含一个变量名（标识符）及其类型，用`:`分隔。多个变量声明用`,`分隔。变量的初始值在合约创建时就要指定。

### 可变变量

可变变量代表着合约的可变状态。它们也被称为字段。它们会在不可变变量之后声明，每个声明都以`field`为前缀。

```
field name1 : type1 = expr1
field name2 : type2 = expr2
...
```

这里的每个表达式都是相应字段的初始值。变量的定义在创建就完成了初始化。随着合约履行transition，这些字段的值会被修改。

### 转换（Transitions）

Transitions决定了合约状态的变化。这些是使用`transition`定义的，后面写要传递的名称和参数，以`end`结尾。

```
transition foo (name1 : type1, name2 : type2, ...)
  ...
end
```

`name : type`指定参数名和类型，多个参数时用`,`分隔。除了显式声明的参数之外，还可以使用以下`transition`隐式参数。

- `_sender : Address` ：触发此transition的帐户（消息发送人）。
- `_amount : Uint128`：入库金额（ZILs）。必须使用`accept`语句明确接收过程。如果transition没有执行，则不会执行`accept`。

### 表达式（Expressions）

表达式处理纯操作。Scilla中支持的表达式是：

- `let x = f in e` ：在表达式`e`中给出名称`x`。 这里`e`与`f`的结合是局部的，因此限于`e`。 下面的示例将表达式`builitin add one Int32 5`中的`one`绑定为`1` ，它将5添加到1，因此计算结果为6。

  > ```
  > let one = 1 in builtin add one Int32 5
  > ```

- `let x = f`：在合约中注明 `f`名称`x`。`x`to 的绑定 `f`是全局的，并延伸到合约的末尾。请注意缺失`in`，这意味着绑定适用于整个合约而不是特定表达式。以下代码片段定义了一个常量，`one`其值`1`贯穿整个合约。

  > ```
  > let one = 1
  > ```

- `{<entry> _1; <entry> _2 ...}`：消息表达式（请参阅下面的`Message`类型），其中每个条目具有以下形式：`b:x`。这里`b`是标识符，`x`是变量，其值绑定到消息中的标识符。以下代码定义了一个带有四个条目`_tag`，`_recipient`，`_amount`和`code的msg`。

  > ```
  > msg = { _tag : "Main"; _recipient : sender; _amount : Uint128 0; code : Uint32 0 };
  > ```

- `fun (x : T) => e`：此函数将类型为`T`的变量`x`输入至表达式`e`中求值。

- `tfun T => e`：一种类型函数，它将`T`作为参数类型并输入至表达式`e里`求值。请参阅后文`Pair`中的示例。

- `@x T`：实例化类型为`T`的变量`x`。

- `f x`：用`f`对`x`进行处理。

- `builtin f x`：用`builtin`方法`f`对`x`进行处理。

- `match`expression：将绑定变量与模式匹配并执行该子句中的语句。该`match`表达式类似于`match`OCaml中的表达式 。要匹配的模式可以是变量绑定，ADT构造函数（请参阅[ADT](./代数数据类型.md)）或通配符`_`以匹配任何内容。

```
match x with
| pattern1 =>
   statements ...
| pattern2 =>
   statements ...
end
```

### 声明（Statements）

Scilla中的声明是实际生效的操作，它们并不是纯数学意义上的操作。此类操作包括从智能合约的可变变量读取或向其写入。

- `x <- f`：`x`从可变字段`f`读取值。
- `f := x`：可变字段`f`的值更新为 `x`。

还可以从区块链状态中读取。区块链状态由与区块相关联的某些值组成，如`BLOCKNUMBER`。

- `x <- &B`：`x`从区块链状态变量`B`读取值。

每当通过transition发送ZIL令牌时，转换必须明确接收这笔转账，可通过`accept`声明完成。

- `accept` ：接收来款。

### 通信（Communication）

合约可以通过`send`声明与其他合约（或非合约）账户进行通信：

- `send ms`：发送消息列表`ms`。

