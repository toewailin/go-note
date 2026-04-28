# 📚 Go Local Modules ချိတ်ဆက်အသုံးပြုခြင်း လမ်းညွှန် (Calling Local Modules)

လုပ်ငန်းခွင်မှာ Go Project တွေ ရေးတဲ့အခါ Package တွေကို သီးခြားခွဲပြီး ရေးသားလေ့ရှိပါတယ်။ ဒီလမ်းညွှန်မှာ မိမိစက် (Local) ထဲမှာပဲရှိနေတဲ့ Go Module နှစ်ခုကို ဘယ်လိုချိတ်ဆက် အသုံးပြုရမလဲဆိုတာကို ရှင်းပြပေးထားပါတယ်။

### 📂 ပရောဂျက် တည်ဆောက်ပုံ (Project Structure)
ငါတို့ဆီမှာ သီးခြားစီ တည်ရှိနေတဲ့ Project Folder ၂ ခု ရှိတယ်ဆိုပါစို့။
```text
<home>/
 |-- greetings/
 |-- hello/
```

---

### ၁။ `greetings` Package တည်ဆောက်ခြင်း

ပထမဦးစွာ `greetings` folder ထဲဝင်ပြီး module အသစ်တစ်ခု စတင်တည်ဆောက်ပါမယ်။

```bash
cd greetings
go mod init example.com/greetings
touch greetings.go
```

**`greetings.go`** ဖိုင်ထဲတွင် အောက်ပါ Code ကို ရေးပါ။
```go
package greetings

import "fmt"

func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

---

### ၂။ `hello` Package မှ လှမ်းခေါ်အသုံးပြုခြင်း

အထက်ပါ `greetings` ရဲ့ package ကို `hello` ထဲမှာ လှမ်းခေါ်ပြီး အသုံးပြုကြည့်ပါမယ်။

```bash
cd ../hello 
go mod init example.com/hello
touch hello.go
```

**`hello.go`** ဖိုင်ထဲတွင် အောက်ပါ Code ကို ရေးပါ။
```go
package main

import (
    "fmt"
    "example.com/greetings"
)

func main() {
    // Get a greeting message and print it.
    message := greetings.Hello("Gladys")
    fmt.Println(message)
}
```

---

### ⚠️ တွေ့ကြုံရမည့် ပြဿနာ (The Error)

အပေါ်က ကုဒ်အတိုင်းသာ run လိုက်မယ်ဆိုရင် Package မတွေ့ဘူးဆိုတဲ့ **Error** တက်လာပါလိမ့်မယ်။

**ဘာဖြစ်လို့လဲဆိုတော့...**
`example.com/greetings` ဆိုတဲ့ package ကို Golang က Internet (ဥပမာ - Github) ပေါ်မှာ သွားရှာနေလို့ ဖြစ်ပါတယ်။ တကယ်တမ်းမှာ ဒီ package က ကိုယ့်စက် (Local) ထဲမှာပဲ ရှိနေတာပါ။ အဲ့ဒီအတွက် Local ထဲက package ကိုပဲ ဆွဲယူအသုံးပြုဖို့ Go Compiler သိအောင် ကိုယ်က သတ်မှတ်ပေးဖို့ လိုအပ်ပါတယ်။

ဖြေရှင်းနိုင်တဲ့ နည်းလမ်း (၂) မျိုး ရှိပါတယ်။

---

### 💡 ဖြေရှင်းနည်း (၁) - `replace` အသုံးပြုခြင်း (Traditional Method)

ကိုယ့်ရဲ့ Terminal ကနေ `hello` folder ထဲကို သွားပြီး အောက်ပါ command ကို ရိုက်ထည့်ပါ။

```bash
cd <home>/hello
go mod edit -replace example.com/greetings=../greetings
```
ဒီ command က `hello/go.mod` ဖိုင်ကို အလိုအလျောက် ပြင်ဆင်ပေးပြီး module path ကို မိမိရဲ့ local လမ်းကြောင်း (`../greetings`) ဖြစ်အောင် ချိတ်ဆက်ပေးလိုက်တာ ဖြစ်ပါတယ်။

ထို့နောက် Dependencies များကို ချိန်ညှိရန် အောက်ပါ command ကို ဆက်သုံးပါ။
```bash
go mod tidy
```

ဒါဆိုရင် `hello/go.mod` file ထဲမှာ အောက်ပါအတိုင်း အလိုအလျောက် ဝင်သွားတာကို တွေ့ရပါလိမ့်မယ်။
```go
module example.com/hello

go 1.24.5

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```

---

### 🚀 ဖြေရှင်းနည်း (၂) - Go Workspaces အသုံးပြုခြင်း (Best Practice for Go 1.18+)

အကယ်၍ သင်က Go Version 1.18 နှင့် အထက်ကို အသုံးပြုနေတယ်ဆိုရင် **Go Workspaces** ဆိုတဲ့ လုပ်ဆောင်ချက်အသစ်ကို အသုံးပြုတာက ပိုပြီး ကောင်းမွန်ပါတယ်။ 

ဒီလုပ်ဆောင်ချက်က `go.mod` ဖိုင်တွေထဲမှာ `replace` တွေကို လိုက်လံပြင်ဆင်နေစရာမလိုဘဲ local module အများအပြားကို တစ်ပြိုင်နက်တည်း ချိတ်ဆက်အသုံးပြုနိုင်ဖို့ ဖန်တီးပေးထားတာ ဖြစ်ပါတယ်။

**အသုံးပြုနည်း -**
ပင်မဖိုင်တွဲဖြစ်တဲ့ `<home>` directory သို့ သွားပြီး အောက်ပါ command ကို ရိုက်ထည့်ပါ။

```bash
cd <home>
go work init ./hello ./greetings
```

ဒီလိုလုပ်လိုက်တာနဲ့ `<home>` folder ထဲမှာ `go.work` ဆိုတဲ့ ဖိုင်တစ်ခုကို အလိုအလျောက် ဖန်တီးပေးသွားမှာ ဖြစ်ပြီး `hello` နဲ့ `greetings` တို့ ချိတ်ဆက်ထားကြောင်းကို Go မှ အလိုအလျောက် သိရှိသွားမှာ ဖြစ်ပါတယ်။
go.work file ထဲမှာ အောက်ပါအတိုင်း တွေ့ရမှာ ဖြစ်ပါတယ်။

```go
go 1.24.5

use (
        ./greetings
        ./hello
)
```
