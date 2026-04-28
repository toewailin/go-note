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

Production (သို့မဟုတ်) Server ပေါ်တင်ပြီး Compile လုပ်တဲ့အခါ သေချာပေါက် ကြုံတွေ့ရနိုင်တဲ့ ပြဿနာ (Errors) တွေ ရှိပါတယ်။ Local မှာ အဆင်ပြေနေပေမယ့် Server ပေါ်ရောက်တဲ့အခါ အဖြစ်များဆုံး ပြဿနာတွေနဲ့ ဖြေရှင်းနည်းတွေကို အောက်မှာ စုစည်းပေးလိုက်ပါတယ်။


### ၁။ Local Path များကို Server က မသိခြင်း (Path/Directory Structure Mismatch)

**ပြဿနာ -** `go.mod` ထဲမှာ `replace example.com/greetings => ../greetings` လို့ ရေးထားတာပဲဖြစ်ဖြစ်၊ `go.work` ကို သုံးထားတာပဲဖြစ်ဖြစ် Server ပေါ်တင်တဲ့အခါ အဲ့ဒီအတိုင်း `../greetings` ဆိုတဲ့ Folder ဖွဲ့စည်းပုံအတိုင်း တိတိကျကျ မရှိရင် `no required module provides package` ဆိုတဲ့ Error ပြန်တက်ပါလိမ့်မယ်။ အထူးသဖြင့် `hello` project တစ်ခုတည်းကိုပဲ Git ပေါ်တင်ပြီး Server ကနေ Pull ဆွဲတဲ့အခါမျိုးမှာ ဖြစ်တတ်ပါတယ်။

**ဖြေရှင်းနည်း -**
* **Monorepo သုံးခြင်း:** Project နှစ်ခုလုံးကို Git Repository တစ်ခုတည်း (Monorepo) ထဲမှာ ထည့်ပြီး Server ပေါ်ကို Folder တွဲလျက် တင်ပါ။
* `go.work` **ကို Git ပေါ် မတင်ပါနဲ့:** `go.work` ဖိုင်က မိမိစက် (Local Machine) အတွက်ပဲ ရည်ရွယ်ပါတယ်။ ဒါကြောင့် `go.work` ဖိုင်ကို `.gitignore` ထဲမှာ ထည့်ထားသင့်ပါတယ်။

### ၂။ OS နဲ့ Architecture မတူညီတဲ့ ပြဿနာ (Cross-Compilation Error)

**ပြဿနာ -**
အကယ်၍ Server ပေါ်မှာ တိုက်ရိုက် Compile မလုပ်ဘဲ ကိုယ့်ရဲ့ Local စက်မှာ `go build` လုပ်ပြီး ထွက်လာတဲ့ Binary File (`.exe` သို့မဟုတ် executable file) ကို Server ပေါ် တိုက်ရိုက် Upload တင်တဲ့အခါမျိုးမှာ အလုပ်မလုပ်ဘဲ `exec format error` ဆိုပြီး တက်တတ်ပါတယ်။
ဥပမာ - ကိုယ်က Apple Silicon (M-series) chip တွေလိုမျိုး ARM64 Architecture သုံးထားတဲ့ စက်ပေါ်မှာ Build လုပ်ပြီး၊ Production Server က Ubuntu (Linux AMD64) ဖြစ်နေတဲ့အခါ မျိုးမှာ OS နဲ့ Architecture မကိုက်ညီလို့ Run လို့မရတာမျိုး ဖြစ်ပါတယ်။

**ဖြေရှင်းနည်း -**
Go မှာ Cross-compilation လုပ်ရတာ အရမ်းလွယ်ကူပါတယ်။ Build လုပ်တဲ့အခါ Target OS နဲ့ Architecture ကို အောက်ပါအတိုင်း သတ်မှတ်ပေးလိုက်ရုံပါပဲ။

```bash
# Linux Server (AMD64) အတွက် Build လုပ်ရန်
GOOS=linux GOARCH=amd64 go build -o hello-app .
```
ဒီလိုလုပ်လိုက်ရင် ထွက်လာတဲ့ `hello-app` binary file က Server ပေါ်မှာ အေးဆေး Run လို့ ရသွားပါပြီ။

### ၃။ Docker အသုံးပြုပါက Context Path မှားယွင်းခြင်း (Docker Build Error)

**ပြဿနာ -**
Production အတွက် Docker ကို အသုံးပြုမယ်ဆိုရင် `Dockerfile` ထဲမှာ `COPY` လုပ်တဲ့အခါ `hello` folder တစ်ခုတည်းကိုပဲ ထည့်ထားရင် `replace` လုပ်ထားတဲ့ `greetings` folder ကို Docker Container ထဲမှာ ရှာမတွေ့ဘဲ Build Error တက်ပါမယ်။

**ဖြေရှင်းနည်း -**
`Dockerfile` ကို Project နှစ်ခုလုံးရဲ့ အပြင်ဘက် (Parent Directory - `<home>`) မှာထားပြီး Folder နှစ်ခုလုံးကို Docker ထဲ ကူးထည့်ပေးမှ ရပါမယ်။

```dockerfile
# Parent directory တွင်ရှိသော Dockerfile ဥပမာ
FROM golang:1.24.5-alpine
WORKDIR /app

# Module နှစ်ခုလုံးကို Copy ကူးထည့်ပါ
COPY greetings/ ./greetings/
COPY hello/ ./hello/

# Hello ထဲသို့ဝင်၍ Build လုပ်ပါ
WORKDIR /app/hello
RUN go build -o main .
CMD ["./main"]
```

### ၄။ Private Repositories ကို ဆွဲယူမရခြင်း (Authentication Issues)

**ပြဿနာ -**
အကယ်၍ `example.com/greetings` ကို Local အနေနဲ့ မဟုတ်ဘဲ Github/Gitlab ပေါ်က Private Repo အဖြစ် တင်ထားခဲ့မယ်ဆိုရင်၊ Server ပေါ်မှာ `go mod tidy` ဒါမှမဟုတ် `go build` လုပ်တဲ့အခါ Authentication မရှိလို့ ဆွဲမရတဲ့ 403 Forbidden Error တက်ပါမယ်။

**ဖြေရှင်းနည်း -**
* Server မှာ Git SSH Keys တွေ သေချာချိတ်ဆက်ထားဖို့ လိုပါတယ်။
* Go ကို Private Repo တွေအတွက် Proxy မကျော်ဖို့ `GOPRIVATE` ကို သတ်မှတ်ပေးရပါမယ်။
    ```bash
    export GOPRIVATE=github.com/your-username/*
    ```

---

VPS ပေါ်သို့ Go Application ကို တိုက်ရိုက် Deploy လုပ်ရာတွင် Professional Standard အတိုင်း အသုံးပြုလေ့ရှိသည့် အကောင်းဆုံးနည်းလမ်းမှာ **Systemd** ကို အသုံးပြု၍ Background Service အဖြစ် Run ခြင်းနှင့် **Nginx** ကို Reverse Proxy အနေဖြင့် အသုံးပြုခြင်း ဖြစ်ပါသည်။ 

Go ၏ အကြီးမားဆုံး အားသာချက်မှာ Compile လုပ်လိုက်ပါက **Executable Binary File တစ်ခုတည်း**သာ ထွက်လာမည်ဖြစ်သောကြောင့် သင်၏ Production VPS Server ပေါ်တွင် Go ကို Install လုပ်ထားရန်ပင် မလိုအပ်တော့ပါ။

အောက်ပါအတိုင်း အဆင့်လိုက် လုပ်ဆောင်ရမည်

---

# 🚀 VPS ပေါ်သို့ Go Application Deploy လုပ်ခြင်း လမ်းညွှန် (Direct Deployment Guide)

**လိုအပ်ချက်များ (Prerequisites) -**
* Linux VPS (ဥပမာ - Ubuntu 22.04 သို့မဟုတ် 24.04) တစ်ခု ရှိရမည်။
* Server သို့ SSH ဝင်နိုင်ရမည်။
* Domain Name တစ်ခု ရှိပါက ပိုကောင်းပါသည်။

---

### အဆင့် (၁) - Local တွင် Server အတွက် Build လုပ်ခြင်း (Cross-Compilation)

Server ပေါ်တွင် Source Code များ သွားတင်ပြီးမှ Compile လုပ်မည့်အစား မိမိစက် (Local) တွင် Server ၏ OS Architecture နှင့် ကိုက်ညီသော Binary ဖိုင်ကို ကြိုတင် Build လုပ်ရပါမည်။ 

Apple Silicon Mac (M-series chip များ) ကဲ့သို့သော ARM64 Architecture အသုံးပြုထားသည့် စက်များမှနေ၍ Production Server အများစုဖြစ်သော Linux AMD64 အတွက် Build လုပ်ရန် သင်၏ Terminal တွင် အောက်ပါ Command ဖြင့် Build ပါ။

```bash
# hello project folder ထဲတွင် အောက်ပါအတိုင်း run ပါ
GOOS=linux GOARCH=amd64 go build -o myapp .
```
*မှတ်ချက် - ဤအဆင့်တွင် `myapp` ဆိုသည့် Binary ဖိုင်တစ်ခု ထွက်လာပါမည်။ Code များထဲတွင် အသုံးပြုထားသော `example.com/greetings` ကဲ့သို့သော local module အားလုံးသည် ဤ `myapp` ဖိုင်ထဲသို့ အလိုအလျောက် ပေါင်းစည်း (Embed) ပြီးသား ဖြစ်သွားပါပြီ။*

---

### အဆင့် (၂) - Server ပေါ်သို့ Binary ဖိုင် Upload တင်ခြင်း

ထွက်လာသော `myapp` ဖိုင်ကို `scp` (Secure Copy) အသုံးပြု၍ သင်၏ VPS ပေါ်သို့ လှမ်းပို့ပါမည်။ Server ပေါ်တွင် သိမ်းဆည်းရန် `/var/www/myapp` ကဲ့သို့သော လမ်းကြောင်းတစ်ခုကို ရွေးချယ်နိုင်ပါသည်။

```bash
# Server ပေါ်တွင် Folder အရင်သွားဆောက်ရန် (SSH မှတဆင့်)
ssh username@your_server_ip "mkdir -p /var/www/myapp"

# Local မှနေ၍ ဖိုင်လှမ်းပို့ရန်
scp myapp username@your_server_ip:/var/www/myapp/
```

ဖိုင်ရောက်သွားပါက Server ထဲသို့ SSH ဝင်ပြီး အဆိုပါဖိုင်ကို Execute လုပ်ခွင့် (Run ခွင့်) ပေးရပါမည်။
```bash
ssh username@your_server_ip
chmod +x /var/www/myapp/myapp
```

---

### အဆင့် (၃) - Systemd Service ဖန်တီးခြင်း (Background တွင် အမြဲ Run ရန်)

Server ပိတ်သွားလျှင်ဖြစ်စေ၊ Restart ကျသွားလျှင်ဖြစ်စေ Go App ကို အလိုအလျောက် ပြန်တက်လာစေရန်နှင့် Background တွင် အမြဲအလုပ်လုပ်နေစေရန် Systemd Service တစ်ခု ဖန်တီးရပါမည်။

**၁။ Service ဖိုင်အသစ်တစ်ခု ဆောက်ပါ။**
```bash
sudo nano /etc/systemd/system/myapp.service
```

**၂။ အောက်ပါ Configuration ကို ထည့်သွင်းပါ။** *(Port 8080 တွင် Run သည်ဟု ယူဆထားပါသည်။)*
```ini
[Unit]
Description=My Go Application
After=network.target

[Service]
User=root
# လုံခြုံရေးအရ root အစား သီးသန့် user တစ်ခုဆောက်ပြီး သုံးရန် အကြံပြုပါသည်။
WorkingDirectory=/var/www/myapp
ExecStart=/var/www/myapp/myapp
Restart=always
RestartSec=5

# ပတ်ဝန်းကျင် ကိန်းရှင်များ (Env Variables) လိုအပ်ပါက ဤနေရာတွင် ထည့်နိုင်သည်
# Environment="PORT=8080"
# Environment="DB_HOST=localhost"

[Install]
WantedBy=multi-user.target
```

**၃။ Service ကို စတင်၍ Enable လုပ်ပါ။**
```bash
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
```
*(ယခုဆိုလျှင် သင်၏ Go App သည် VPS ပေါ်တွင် အမြဲတမ်း Run နေပြီ ဖြစ်ပါသည်။ အခြေအနေကို ကြည့်လိုပါက `sudo systemctl status myapp` ဖြင့် စစ်ဆေးနိုင်ပါသည်။)*

---

### အဆင့် (၄) - Nginx ကို Reverse Proxy အဖြစ် သတ်မှတ်ခြင်း

Go App ကို Port `8080` (သို့မဟုတ် အခြား Port) တွင် Run ထားသော်လည်း၊ အသုံးပြုသူများက Domain (Port `80`/`443`) မှ ဝင်ရောက်သည့်အခါ အဆိုပါ Go App ထံသို့ Nginx မှတဆင့် ပြန်လည်လမ်းကြောင်းလွှဲ (Reverse Proxy) ပေးရန် လိုအပ်ပါသည်။

**၁။ Nginx Install လုပ်ပါ။**
```bash
sudo apt update
sudo apt install nginx
```

**၂။ Nginx Configuration ဖိုင်အသစ် ဆောက်ပါ။**
```bash
sudo nano /etc/nginx/sites-available/myapp
```

**၃။ အောက်ပါအတိုင်း ရေးထည့်ပါ။**
```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com; # မိမိ Domain အမည်ပြောင်းပါ သို့မဟုတ် IP ထည့်ပါ

    location / {
        proxy_pass http://localhost:8080; # Go App ၏ Port နှင့် ကိုက်ညီရပါမည်
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**၄။ Configuration ကို ချိတ်ဆက်ပြီး Nginx ကို Restart ချပါ။**
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

### အဆင့် (၅) - Firewall ဖွင့်ခြင်း (UFW)

နောက်ဆုံးအနေဖြင့် Server သို့ အပြင်ဘက်မှ ဝင်ရောက်နိုင်ရန် သက်ဆိုင်ရာ Port များကို ဖွင့်ပေးပါ။
```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

🎉 **ပြီးပါပြီ။** ယခုဆိုလျှင် သင်၏ Go Application သည် VPS ပေါ်တွင် Professional အဆင့်မီ လုံခြုံစိတ်ချစွာ အလုပ်လုပ်နေပြီ ဖြစ်ပါသည်။ နောင်တစ်ချိန် Code အသစ်များ Update လုပ်လိုပါက Local တွင် `go build` အသစ်ပြန်လုပ်၊ Server ပေါ်ရှိ `/var/www/myapp/` ထဲသို့ overwrite ဝင်လုပ်ပြီး `sudo systemctl restart myapp` ဟု ရိုက်ပေးလိုက်ရုံသာ ဖြစ်ပါသည်။

---

Production အဆင့် (Enterprise level) တွေမှာဆိုရင်တော့ Manual သွားတင်တာထက် Docker နဲ့ CI/CD (Continuous Integration / Continuous Deployment) ကို အသုံးပြုတာက အကောင်းဆုံး (Best Practice) ဖြစ်ပါတယ်။ 

ဒီနည်းလမ်းက Code ကို Github ပေါ် Push လုပ်လိုက်တာနဲ့ Server ပေါ်မှာ အလိုအလျောက် Update သွားဖြစ်မယ့် (Automated Pipeline) စနစ် ဖြစ်ပါတယ်။ 

အောက်မှာ သင့်ရဲ့ `hello` နဲ့ `greetings` Project အတွက် Docker + Github Actions အသုံးပြုပြီး Deploy လုပ်မယ့် Professional Documentation ကို ရေးသားပေးလိုက်ပါတယ်။

---

# 🐳 Docker နှင့် CI/CD အသုံးပြု၍ Go App Deploy လုပ်ခြင်း လမ်းညွှန် (Modern DevOps Guide)

**အလုပ်လုပ်မည့် ပုံစံ (Architecture) -**
၁။ သင်က Code ကို Github သို့ Push လုပ်မည်။
၂။ Github Actions (CI) မှ Code ကို ယူ၍ Docker Image တည်ဆောက်ပြီး Docker Hub သို့ ပို့မည်။
၃။ Github Actions (CD) မှ သင်၏ VPS Server သို့ SSH ဖြင့် လှမ်းဝင်ပြီး အသစ်ရောက်လာသော Docker Image ကို Pull ဆွဲကာ Run ပေးမည်။

---

### အဆင့် (၁) - Multi-stage Dockerfile တည်ဆောက်ခြင်း

Go App များအတွက် Dockerize လုပ်ရာတွင် **Multi-stage Build** ကို အသုံးပြုခြင်းက အကောင်းဆုံးဖြစ်ပါတယ်။ ၎င်းက Build လုပ်ရန် လိုအပ်သည့် ဖိုင်ကြီးများကို ဖယ်ရှားပေးပြီး နောက်ဆုံးထွက်လာမည့် Docker Image ကို (ဥပမာ - ~1GB မှ ~15MB သို့) အလွန်သေးငယ်သွားစေပါတယ်။

သင့် Project ၏ ပင်မဖိုင်တွဲ (Parent Directory - `<home>`) တွင် `Dockerfile` ဟူသော အမည်ဖြင့် ဖိုင်တစ်ခု ဆောက်ပြီး အောက်ပါ Code များကို ထည့်ပါ။

```dockerfile
# Stage 1: Build Stage (Go ကို အသုံးပြု၍ Compile လုပ်ခြင်း)
FROM golang:1.24.5-alpine AS builder

# လုပ်ငန်းခွင် လမ်းကြောင်း သတ်မှတ်ခြင်း
WORKDIR /app

# go.work အသုံးပြုထားပါက ပရောဂျက်တစ်ခုလုံးကို Copy ကူးထည့်ရန် လိုအပ်ပါသည်
COPY . .

# hello folder ထဲသို့ဝင်၍ Compile လုပ်ပါ
WORKDIR /app/hello
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# ---------------------------------------------------

# Stage 2: Run Stage (အလွန်သေးငယ်သော Alpine Linux ဖြင့် Run ခြင်း)
FROM alpine:latest

WORKDIR /root/

# Stage 1 မှ ထွက်လာသော 'myapp' binary ကိုသာ ဆွဲယူပါမည်
COPY --from=builder /app/hello/myapp .

# Nginx သို့မဟုတ် အပြင်ဘက်သို့ ဖွင့်ပေးမည့် Port
EXPOSE 8080

# Application ကို စတင် Run ပါမည်
CMD ["./myapp"]
```

---

### အဆင့် (၂) - Server ပေါ်တွင် `docker-compose.yml` ဖန်တီးခြင်း

VPS Server ပေါ်တွင် Docker Image ကို အလွယ်တကူ စီမံနိုင်ရန် `docker-compose.yml` ကို အသုံးပြုပါမည်။ သင်၏ VPS Server ပေါ်သို့ SSH ဝင်ပြီး Application ထားရှိမည့် နေရာ (ဥပမာ - `/var/www/mygoapp`) တွင် အောက်ပါဖိုင်ကို ဖန်တီးပါ။

```yaml
# /var/www/mygoapp/docker-compose.yml
version: '3.8'

services:
  app:
    image: your_dockerhub_username/mygoapp:latest
    container_name: go_hello_app
    ports:
      - "8080:8080" # VPS ရဲ့ Port 8080 ကို Container ရဲ့ 8080 နဲ့ ချိတ်ဆက်ခြင်း
    restart: always # Server ပိတ်သွားလျှင် ပြန်တက်လာစေရန်
```

*(မှတ်ချက် - VPS တွင် Docker နှင့် Docker Compose ကို Install လုပ်ထားရန် လိုအပ်ပါသည်။)*

---

### အဆင့် (၃) - Github Repository တွင် Secrets များ ထည့်သွင်းခြင်း

CI/CD မှ သင်၏ Server နှင့် Docker Hub သို့ လှမ်းချိတ်နိုင်ရန် လျှို့ဝှက်အချက်အလက် (Secrets) များကို Github တွင် ထည့်ထားရပါမည်။
သင်၏ Github Repository > **Settings** > **Secrets and variables** > **Actions** သို့သွား၍ အောက်ပါတို့ကို "New repository secret" အဖြစ် ထည့်ပါ။

* `DOCKER_USERNAME` : သင်၏ Docker Hub username
* `DOCKER_PASSWORD` : သင်၏ Docker Hub Password (သို့မဟုတ် Access Token)
* `SERVER_HOST` : သင်၏ VPS Server IP (ဥပမာ - 192.168.1.100)
* `SERVER_USER` : VPS သို့ ဝင်မည့် username (ဥပမာ - root သို့မဟုတ် ubuntu)
* `SERVER_SSH_KEY` : VPS ၏ Private Key (Cat `~/.ssh/id_rsa` ဖြင့် ထုတ်ယူနိုင်သည်)

---

### အဆင့် (၄) - CI/CD Pipeline ရေးသားခြင်း (Github Actions)

ယခု Code များကို အလိုအလျောက် Deploy လုပ်ပေးမည့် အလုပ်သမား (Pipeline) ကို တည်ဆောက်ပါမည်။
သင်၏ Local Project ပင်မဖိုင်တွဲထဲတွင် `.github/workflows/deploy.yml` ဟူသော ဖိုင်လမ်းကြောင်းကို တည်ဆောက်ပြီး အောက်ပါ Code ကို ရေးထည့်ပါ။

```yaml
name: Deploy Go App to VPS

# Main branch သို့ Code များ Push လိုက်တိုင်း ဤ Action အလုပ်လုပ်မည်
on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and Push Docker Image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/mygoapp:latest

    - name: Deploy to VPS Server via SSH
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SERVER_SSH_KEY }}
        script: |
          cd /var/www/mygoapp
          # နောက်ဆုံးထွက် Image အသစ်ကို Docker Hub မှ ဆွဲယူမည်
          docker pull ${{ secrets.DOCKER_USERNAME }}/mygoapp:latest
          # အဟောင်းကို ပိတ်ပြီး အသစ်ဖြင့် ပြန် Run မည်
          docker-compose down
          docker-compose up -d
          # မလိုအပ်တော့သော Image အဟောင်းများကို ရှင်းလင်းမည်
          docker image prune -f
```

---

### အကျဉ်းချုပ် အလုပ်လုပ်ပုံ (The Workflow in Action)

အထက်ပါအဆင့်များ ပြီးစီးသွားပါက သင် Code အသစ်ပြင်ပြီးတိုင်း အောက်ပါအတိုင်း Git သို့သာ Push လုပ်လိုက်ပါ။

```bash
git add .
git commit -m "Update greeting message"
git push origin main
```

ထိုအခါ **Github Actions** မှ အလိုအလျောက် အလုပ်စတင်မည် ဖြစ်ပြီး 
၁။ Code အသစ်ကို ယူကာ `Dockerfile` အတိုင်း Docker Image အသစ် ပြောင်းလဲတည်ဆောက်ပေးမည်။
၂။ ထွက်လာသော Image ကို Docker Hub သို့ သိမ်းဆည်းပေးမည်။
၃။ သင့် VPS Server ထဲသို့ SSH ဖြင့် အလိုအလျောက် ဝင်ရောက်ပြီး `docker-compose down` ဖြင့် အဟောင်းကို ရပ်ကာ၊ `docker pull` နှင့် `docker-compose up -d` ကို အသုံးပြု၍ Application အသစ်ကို စက္ကန့်ပိုင်းအတွင်း ပြန်လည် Run ပေးသွားမည် ဖြစ်ပါသည်။ (Zero Downtime နီးပါး ရရှိမည် ဖြစ်သည်)။
