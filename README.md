# basher 🐚

A simple bash key-value store that lets you save, retrieve, and export variables directly into your shell session.

  ██████╗  █████╗ ███████╗██╗  ██╗███████╗██████╗ 
  ██╔══██╗██╔══██╗██╔════╝██║  ██║██╔════╝██╔══██╗
  ██████╔╝███████║███████╗███████║█████╗  ██████╔╝
  ██╔══██╗██╔══██║╚════██║██╔══██║██╔══╝  ██╔══██╗
  ██████╔╝██║  ██║███████║██║  ██║███████╗██║  ██║
  ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝

it's 3am. you have 47 tabs open. you forgot the target IP again... (╯°□°)╯

 (ง •̀_•́)ง  <-- you, aggressively running to basher

  typing IPs     remembering IPs    using basher
     :-|              :-D              (⌐■_■)
    /|  \            /|  \            /|   \
   / \   \          / \   \          / \    \
   
 [suffering]        [coping]        [ascended]
     ┌──────────────────────┐
     │ target = 10.10.10.6  │
     │ user   = admin       │
     │ pass   = Passw0rd!   │
     │ port   = 445         │
     └──────────────────────┘

     $ nmap $target              # didn't type the IP
     $ ssh $user@$target         # didn't type the user
     $ evil-winrm -i $target -u $user -p $pass      

---

## ⚠️ Important: How to Run

Because `basher` needs to export variables into your **current shell**, it must always be run with `source`:

```bash
source ./basher target 10.10.10.6
echo $target
# → 10.10.10.6
```

Running it as `./basher` won't work for exports — the variables would be set in a subshell and lost immediately.

### 💡 Tip: Create an alias so you never have to type `source`

Add this to your `~/.bashrc`:

```bash
alias basher='source ~/basher'
```

Then reload your shell:

```bash
source ~/.bashrc
```

Now you can use it naturally:

```bash
basher target 10.10.10.6
echo $target
# → 10.10.10.6
```

---

## Usage

### Set a variable
Saves the variable to disk **and** exports it into the current shell immediately.

```bash
source ./basher target 10.10.10.6
source ./basher user myuser
source ./basher port 8080
```

### Get a variable
```bash
source ./basher target
# → 10.10.10.6
```

### List all stored variables
```bash
source ./basher --list
# VARIABLE             VALUE
# --------             -----
# target               10.10.10.6
# user                 myuser
# port                 8080
```

### Load all variables into a new shell session
Variables are saved to disk, so after opening a new terminal you can restore them all:

```bash
source ./basher --load
# Exported: target = 10.10.10.6
# Exported: user = myuser
# Exported: port = 8080
```

### Delete a variable
Removes the variable from disk and unsets it from the current shell.

```bash
source ./basher --delete port
```

### Clear all variables
```bash
source ./basher --clear
```

---

## Storage

Variables are stored in `~/.basher_store` as plain `KEY=VALUE` pairs:

```
target=10.10.10.6
user=myuser
port=8080
```

You can override the storage location with the `BASHER_STORE` environment variable:

```bash
export BASHER_STORE=/tmp/my_custom_store
source ./basher target 10.10.10.6
```

---

## Real-world example

```bash
source ./basher target 10.10.10.6
source ./basher user myuser

# Use variables in other tools
nmap -sV $target
ssh $user@$target
curl http://$target

./basher --list
VARIABLE             VALUE
--------             -----
target               10.10.10.6
user                 th3b3stus3r3v3r
```

---

## Variable naming rules

Variable names must start with a letter or underscore, and contain only letters, digits, and underscores.

```bash
source ./basher my_var value      # ✅ valid
source ./basher 1var value        # ❌ invalid
source ./basher my-var value      # ❌ invalid
```
