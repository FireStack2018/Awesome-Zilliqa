# Scilla入门

## Hello World

我们先来编写一份符合以下规范的经典`HelloWorld.scilla`合约：

- 有一个由合约的创建者初始化的不变变量 `owner`。该变量初始化后，其值就不能改变。`owner`的类型是 `Address`。
- 有一个初始化为`""`的可变变量 `welcome_msg`，该变量为`String`类型。这里的可变性是指即使在部署合约之后也可以修改变量的值。
- 只有**`owner`**才能通过接口`setHello`修改`welcome_msg`。该接口需要`msg`（`String`类型）作为输入方，并允许`owner`设置发送给`msg`的 `welcome_msg`的值。
- 有一个`getHello`接口可获取到`welcome_msg`。`getHello`不接收任何内容输入。

### 定义合约及（不）可变字段

使用`contract`声明合约，让合约正式生效。`contract`后面跟着合约名称，如例子中的`HelloWorld`。因此，以下代码声明了一个`HelloWorld`合约。

```
contract HelloWorld
```

> 注意
>
> 目前，Scilla合约只能包含单个合约声明，因此`contract` 后面的任何字符都是合约声明的一部分。换句话说，没有明确的关键字来声明合约定义的结束点。

合约声明后，接着要声明不可变变量，由`()`来定义。每个不可变变量以下列方式声明：`vname: vtype`，其中`vname`是变量名称，`vtype`是变量类型。不可变变量用`,`分隔。根据规范，合约将只有一个不可变的类型变量`owner`，类型为`Address`。参阅如下代码。

```
(owner: Address)
```

合约中的可变变量通过`field`声明。每个可变变量都按以下方式声明：`field vname : vtype = init_val`，`vname`是变量名称，`vtype`是它的类型，`init_val` 是变量的初始值。`HelloWorld`合约带有一个可变参数`welcome_msg`（`String`类型），初始化为`""`。代码如下：

```
field welcome_msg : String = ""
```

至此，`HelloWorld.scilla`合约形式如下，其中包含合约名称及其可变/不可变变量：

```
contract HelloWorld
(owner: Address)

field welcome_msg : String = ""
```

> 注意
>
> 除了这些字段之外，Scilla中的合约都有一个隐式声明的可变字段`_balance`（在合约创建时初始化），这代表了合约范围内的资金量。该字段可在合约过程中免费读取，但只能在显式转账时才能修改。

### 定义接口（即Transitions）

类似`setHello`的接口在Scilla中称为*transitions*。Transitions类似于其他语言中的函数或方法。

> 注意
>
> 术语transition来自Scilla的通信自动机里的基础计算模型。Scilla的合约是一种带有状态的自动机。自动机的状态可以通过transition来改变，即transition接收先前的状态和输入信息，然后变为新的状态。查看[维基百科条目](https://en.wikipedia.org/wiki/Transition_system) 以了解transition系统的更多信息。

transition使用`transition`来声明。transition用`end`表示结束。`transition` 后面为transition名称，如例子中的`setHello`。名称后面是用`()`括起来的参数。每个参数用`,`分隔，声明格式如下：`vname : vtype`。按照规范，`setHello`只需要一个`string`类型的`msg`参数。代码如下：

```
transition setHello (msg : String)
```

然后是transition的主体。例如，在transition`setHello (msg :  String)`中设定`welcome_msg`，代码如下：

```
transition setHello (msg : String)
  is_owner = builtin eq owner _sender;
  match is_owner with
  | False =>
    msg = {_tag : "Main"; _recipient : _sender; _amount : Uint128 0; code : not_owner_code};
    msgs = one_msg msg;
    send msgs
  | True =>
    welcome_msg := msg;
    msg = {_tag : "Main"; _recipient : _sender; _amount : Uint128 0; code : set_hello_code};
    msgs = one_msg msg;
    send msgs
  end
end
```

首先看代码第二行，用`builtin eq owner _sender`来判断`owner`是否正确。为了比较两个账户地址，我们使用了`builtin`方法的`eq`。这返回布尔值`True`或`False`。

> 注意
>
> Scilla在内部定义了一些具有特殊语义的变量。这些特殊变量通常以`_`为前缀。例如，`_sender`指的是当前合约调用的帐户地址。

根据比较结果，transition根据匹配结果进入不同的分支路径，语法如下：

```
match expr with
| x => expr_1
| y => expr_2
end
```

上面代码检查`expr`是符合`x`还是`y`。如果`expr` 符合`x`，那么下一步将是`expr_1`，如果`expr` 符合`y`，那么下一步将是`expr_2`。简单地说，上面的代码实现了一条 `if-then-else`指令。

#### 调用者不是owner的情况

如果调用者不是`owner`，则transition会走 `False`分支，此时合约会输出一条消息。Scilla为输出消息定义了一种特殊类型 `Message`。输出消息包含需要调用的其他合约信息（作为当前调用的一部分）或需要返回的值。

在这种情况下，输出消息是`msg`中的一个错误代码`not_owner_code`。具体来说，这种情况下的输出消息是：

```
msg = {_tag : "Main"; _recipient : _sender; _amount : Uint128 0; code : not_owner_code};
```

输出消息参数格式为`vname : value`组成，用`;`分隔，其范围在`{}`之内。每个输出消息必须有三个必填字段：`_tag`、 `_recipient` 、`_amount`，字段顺序无要求。`_recipient`是消息发送地址。`_tag`是`_recipient`里要调用的transition的名称，`_amount`是要转移给`_recipient`的ZIL数量。

除了这些必填字段，消息可能还有其他字段。在这个示例中，消息使用`code`字段来报告错误消息。

`send`指令能承载一系列 `Message`类型的内容，因此会用它来发送消息。在当前示例中，项内容仅有一个消息。总而言之，以下代码将创建并发送一条消息。

```
msgs = one_msg msg;
send msgs
```

`one_msg`是一个实用函数，它允许创建消息列表并插入至`msg`列表中。

#### 调用者是owner的情况

如果调用者是`owner`，则合约允许调用者设定可变变量`welcome_msg`的值，并作为`msg`的输入参数。代码如下：

```
welcome_msg := msg;
```

> 注意
>
> 写入可变参数是通过`:=`完成的。

在上述例子中，合约使用`set_hello_code`向调用者发送了一条消息。

### 库

Scilla合约可能带有一些辅助库，包含一些合约的纯功能（没有状态管理）组件。库会在合约的最前面使用`library`加上库名称来声明。库声明如下例所示：

```
library HelloWorld
```

在这个示例中，库将使用标准用法`let x = y in expr`来定义错误代码。

```
let not_owner_code  = Int32 1
let set_hello_code  = Int32 2
```

该库还可以包括实用函数。例如，`one_msg`函数创建了一个包含`Message`类型项的列表 ，如下所示：

```
let one_msg =
   fun (msg : Message) =>
   let nil_msg = Nil {Message} in
   Cons {Message} msg nil_msg
```

在此阶段，合约代码如下：

```
library HelloWorld

 let one_msg =
     fun (msg : Message) =>
     let nil_msg = Nil {Message} in
     Cons {Message} msg nil_msg

 let not_owner_code  = Int32 1
 let set_hello_code  = Int32 2


 contract HelloWorld
 (owner: Address)

 field welcome_msg : String = ""

 transition setHello (msg : String)
   is_owner = builtin eq owner _sender;
   match is_owner with
   | False =>
     msg = {_tag : "Main"; _recipient : _sender; _amount : Uint128 0; code : not_owner_code};
     msgs = one_msg msg;
     send msgs
   | True =>
     welcome_msg := msg;
     msg = {_tag : "Main"; _recipient : _sender; _amount : Uint128 0; code : set_hello_code};
     msgs = one_msg msg;
     send msgs
   end
 end
```

### 最后

现在可以添加第二个transition`getHello()`，让所有调用者都能收到`welcome_msg`。声明方法类似`setHello (msg : String)`，而`getHello()`不用带任何参数。

```
transition getHello ()
    r <- welcome_msg;
    msg = {_tag : Main; _recipient : _sender; _amount : 0; msg : r};
    msgs = one_msg msg;
    send msgs
end
```

> 注意
>
> 将内容传入可变变量的操作符为`<-`。即示例中的`r <- welcome_msg`。

下面给出了理想的完整合约范式：

```
(* HelloWorld contract *)


(***************************************************)
(*               Associated library                *)
(***************************************************)
library HelloWorld

let one_msg =
  fun (msg : Message) =>
  let nil_msg = Nil {Message} in
  Cons {Message} msg nil_msg

let not_owner_code  = Int32 1
let set_hello_code  = Int32 2

(***************************************************)
(*             The contract definition             *)
(***************************************************)

contract HelloWorld
(owner: Address)

field welcome_msg : String = ""

transition setHello (msg : String)
  is_owner = builtin eq owner _sender;
  match is_owner with
  | False =>
    msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : not_owner_code};
    msgs = one_msg msg;
    send msgs
  | True =>
    welcome_msg := msg;
    msg = {_tag : "Main"; _recipient : _sender; _amount : 0; code : set_hello_code};
    msgs = one_msg msg;
    send msgs
  end
end

transition getHello ()
    r <- welcome_msg;
    msg = {_tag : Main; _recipient : _sender; _amount : 0; msg : r};
    msgs = one_msg msg;
    send msgs
end
```

## 众筹

本节将介绍一个简单众筹活动的合约。在这个众筹中，项目发起人希望通过社区捐款筹集资金。

假设发起人（`owner`）希望在预定周期（`max_block`）内运行该活动。发起人还希望能设定一个最低筹集金额（`goal`），否则项目无法启动。因此合约有三个不可变的变量`owner`， `max_block`和`goal`。

如果所有者在规定时间内募集到目标金额，则该众筹被视为成功。如果众筹不成功，则捐款将会退还给所有项目支持者。合约包含两个可变变量：捐献者的账户地址和捐献金额组成的键值对`backer`，以及一个布尔值标志`funded`，指示在活动结束后发起人是否已获得了资金。

合约包含三个transition：捐献者为众筹捐款的方法`Donate ()`，**只允许发起人`owner`**宣布众筹金额且获取金额的方法`GetFunds ()`，允许捐献者在活动不成功时索回捐款的方法`ClaimBack()`。

完整的合约如下：

```
(***************************************************)
(*               Associated library                *)
(***************************************************)
library Crowdfunding

let andb =
  fun (b : Bool) =>
  fun (c : Bool) =>
    match b with
    | False => False
    | True  =>
      match c with
      | False => False
      | True  => True
      end
    end

let orb =
  fun (b : Bool) => fun (c : Bool) =>
    match b with
    | True  => True
    | False =>
      match c with
      | False => False
      | True  => True
      end
    end

let negb = fun (b : Bool) =>
  match b with
  | True => False
  | False => True
  end

let one_msg =
  fun (msg : Message) =>
    let nil_msg = Nil {Message} in
    Cons {Message} msg nil_msg

let check_update =
  fun (bs : Map Address Int) =>
  fun (_sender : Address) =>
  fun (_amount : Int) =>
    let c = builtin contains bs _sender in
    match c with
    | False =>
      let bs1 = builtin put bs _sender _amount in
      Some {Map Address Int} bs1
    | True  => None {Map Address Int}
    end

let blk_leq =
  fun (blk1 : BNum) =>
  fun (blk2 : BNum) =>
    let bc1 = builtin blt blk1 blk2 in
    let bc2 = builtin eq blk1 blk2 in
    orb bc1 bc2

let accepted_code = 1
let missed_deadline_code = 2
let already_backed_code  = 3
let not_owner_code  = 4
let too_early_code  = 5
let got_funds_code  = 6
let cannot_get_funds  = 7
let cannot_reclaim_code = 8
let reclaimed_code = 9

(***************************************************)
(*             The contract definition             *)
(***************************************************)
contract Crowdfunding

(*  Parameters *)
(owner     : Address,
 max_block : BNum,
 goal      : Int)

(* Mutable fields *)
field backers : Map Address Int = Emp Address Int
field funded : Bool = False

transition Donate ()
  blk <- & BLOCKNUMBER;
  in_time = blk_leq blk max_block;
  match in_time with
  | True  =>
    bs  <- backers;
    res = check_update bs _sender _amount;
    match res with
    | None =>
      msg  = {_tag : Main; _recipient : _sender; _amount : 0;
              code : already_backed_code};
      msgs = one_msg msg;
      send msgs
    | Some bs1 =>
      backers := bs1;
      accept;
      msg  = {_tag : Main; _recipient : _sender; _amount : 0;
              code : accepted_code};
      msgs = one_msg msg;
      send msgs
    end
  | False =>
    msg  = {_tag : Main; _recipient : _sender; _amount : 0;
            code : missed_dealine_code};
    msgs = one_msg msg;
    send msgs
  end
end

transition GetFunds ()
  is_owner = builtin eq owner _sender;
  match is_owner with
  | False =>
    msg  = {_tag : Main; _recipient : _sender; _amount : 0;
            code : not_owner_code};
    msgs = one_msg msg;
    send msgs
  | True =>
    blk <- & BLOCKNUMBER;
    in_time = blk_leq blk max_block;
    c1 = negb in_time;
    bal <- balance;
    c2 = builtin lt bal goal;
    c3 = negb c2;
    c4 = andb c1 c3;
    match c4 with
    | False =>
      msg  = {_tag : Main; _recipient : _sender; _amount : 0;
              code : cannot_get_funds};
      msgs = one_msg msg;
      send msgs
    | True =>
      tt = True;
      funded := tt;
      msg  = {_tag : Main; _recipient : owner; _amount : bal;
              code : got_funds_code};
      msgs = one_msg msg;
      send msgs
    end
  end
end

(* transition ClaimBack *)
transition ClaimBack ()
  blk <- & BLOCKNUMBER;
  after_deadline = builtin blt max_block blk;
  match after_deadline with
  | False =>
    msg  = {_tag : Main; _recipient : _sender; _amount : 0;
        code : too_early_code};
    msgs = one_msg msg;
    send msgs
  | True =>
    bs <- backers;
    bal <- _balance;
    (* Goal has not been reached *)
    f <- funded;
    c1 = builtin lt bal goal;
    c2 = builtin contains bs _sender;
    c3 = negb f;
    c4 = andb c1 c2;
    c5 = andb c3 c4;
    match c5 with
    | False =>
      msg  = {_tag : Main; _recipient : _sender; _amount : 0;
              code : cannot_reclaim_code};
      msgs = one_msg msg;
      send msgs
    | True =>
      res = builtin get bs _sender;
      match res with
      | None =>
        msg  = {_tag : Main; _recipient : _sender; _amount : 0;
                code : cannot_reclaim_code};
        msgs = one_msg msg;
        send msgs
      | Some v =>
        bs1 = builtin remove bs _sender;
        backers := bs1;
        msg  = {_tag : Main; _recipient : _sender; _amount : v;
                code : reclaimed_code};
        msgs = one_msg msg;
        send msgs
      end
    end
  end
end
```
