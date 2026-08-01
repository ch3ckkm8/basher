# basher 🐚

A simple bash key-value store that lets you save, retrieve, and export variables directly into your shell session.
```
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
```

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

### Set a variable with a custom table column name
An optional third argument lets you give the variable a different display name when it shows up in `--table` (see below). It must be wrapped in square brackets.

```bash
source ./basher target 10.10.10.6 [IP]
source ./basher pass Passw0rd! [Password]
```

If you don't provide a `[ColumnName]`, the table just uses the variable name itself (e.g. `user` above).

```bash
source ./basher foo bar BADCOL
# Error: column name must be wrapped in brackets, e.g. [ColumnName]
```

### Get a variable
```bash
source ./basher target
# → 10.10.10.6
```

### List all stored variables
```bash
source ./basher --list
# VARIABLE             VALUE                COLUMN
# --------             -----                ------
# target               10.10.10.6           IP
# user                 myuser               (default)
# port                 8080                 (default)
```

### Display all variables as a table
Prints all stored variables as a single table, where each **column header is the variable's name** (or its custom `[ColumnName]`, if one was set) and the value sits underneath it.

```bash
source ./basher --table
# IP         user    port
# ---------- ------- ----
# 10.10.10.6 myuser  8080
```

This also automatically writes the same table as a Markdown file — `basher_table.md` — saved in the same directory as the `basher` script itself. It's overwritten every time you run `--table`.

```md
| IP | user | port |
| --- | --- | --- |
| 10.10.10.6 | myuser | 8080 |
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

Variables are stored in `~/.basher_store`, one per line, as `KEY=VALUE` followed by a tab-separated custom column name (which may be empty if you didn't set one):

```
target=10.10.10.6	IP
user=myuser	
port=8080	
```

You don't need to edit this file by hand — it's managed for you by the `set`, `--delete`, and `--clear` commands.

You can override the storage location with the `BASHER_STORE` environment variable:

```bash
export BASHER_STORE=/tmp/my_custom_store
source ./basher target 10.10.10.6
```

## Variable naming rules

Variable names must start with a letter or underscore, and contain only letters, digits, and underscores.

```bash
source ./basher my_var value      # ✅ valid
source ./basher 1var value        # ❌ invalid
source ./basher my-var value      # ❌ invalid
```

Custom table column names (the optional `[ColumnName]` argument) must be wrapped in square brackets, but otherwise have no character restrictions.

---

## Real-world example

```bash
# import entries via commands below:
source basher ip1 10.10.18.3 [IP]
source basher host1 linuxhost [Hostname]
source basher notes1 bob-sworkstation [Notes]
source basher ports1 80,9000 [Ports]
source basher user1 b0b [notable]
source basher pass1 p4ssw0rd [notable]
source basher creds1 $user1:$pass1 [Credentials]

> source basher --list
VARIABLE             VALUE                COLUMN
--------             -----                ------
ip1                  10.10.18.3           IP
user1                b0b                  (excluded from --table)
pass1                p4ssw0rd             (excluded from --table)
host1                linuxhost            Hostname
notes1               bob-sworkstation     Notes
ports1               80,9000              Ports
creds1               b0b:p4ssw0rd         Credentials

> source basher --table

IP         Hostname  Notes            Ports   Credentials  
---------- --------- ---------------- ------- ------------ 
10.10.18.3 linuxhost bob-sworkstation 80,9000 b0b:p4ssw0rd 

Markdown table written to: /home/ch3ckm8/HTB/garfield/basher/basher_table.md

```

---
## Use it along with wyrmgaze

These variables can also be easily be imported to my attack chain visualization tool https://github.com/ch3ckkm8/wyrmgaze
For example, below i import some of the variables i created with basher above towards wyrmgaze:

```bash
echo "$ip1 | *nmap* | $ports1" > test.txt 
echo "$ports1 | *enum* | webapp" >> test.txt

└─$ cat test.txt                                                                                                                                                   
10.10.18.3 | *nmap* | 80,9000
80,9000 | *enum* | webapp

python wyrmgaze.py test.txt

⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣀⡤⢴⣾⠟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡤⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⡤⠴⠚⠉⢀⡴⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡤⣾⠟⠀⠀⣀⣤⣾⣿⣟⣁⠤⠴⠒⠋⠉⠀⠀⢀⣠⠞⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⠴⢚⣁⣴⠧⠔⢚⣹⠿⠟⠛⠉⠁⠀⠀⠀⠀⠀⢀⣠⠖⠋⠀⠀⠀⠀⠄⠀⠀⠀⠀⠀⠀⣀⣀⣀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⠴⠚⣩⠔⠋⠉⠀⠀⠐⠊⠉⠀⢀⠀⠀⠀⠀⠀⠀⢀⣠⠔⠛⠛⠋⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⣉⡭⠟⠛⠉⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡤⠞⠋⣟⡡⠶⠛⠒⠚⠉⠀⠀⠀⠀⠀⠀⡰⠋⠀⠀⠀⠀⢠⡖⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣀⡤⠴⠒⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣀⡤⠖⠋⣀⣴⣋⠥⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⠞⠀⣀⠤⠖⠒⠒⠿⢤⣀⡀⠀⠀⠀⠀⠀⣠⠴⠚⠛⠓⠦⠤⢤⣀⣀⣀⡀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⢠⡖⠋⠀⣠⠖⠋⠁⠀⠀⠀⢀⡀⠀⠀⠀⠀⢀⣀⠤⠞⠁⠀⠉⠀⠀⠀⠀⢀⣀⣀⠀⠉⠙⠒⠦⢴⣋⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣈⣭⠽⠿⠟⠓⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⢀⣸⣧⠀⢸⡁⠀⠀⣠⣴⣲⣶⡏⠀⢀⡠⠖⠊⠉⠀⠀⠀⠀⠀⠀⠀⣼⡟⠉⠁⠀⠈⠉⠉⠒⠲⢤⣀⠈⠙⠲⢤⣀⣀⡤⠴⠶⠯⣅⣀⠀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⣀⣠⡴⠒⠉⠁⠀⢀⡤⠛⠓⠋⠙⠻⠿⠋⢀⡴⠋⢀⡤⠒⠒⠤⣄⣠⡀⠀⠀⠀⣯⠙⠦⣄⠀⠀⠀⠀⠀⠀⠀⠈⠙⠲⠤⣀⣈⣉⣓⣦⣄⡀⠀⠀⠉⠓⠦⣄⡀⠀⠀⠀                                                                                                  
⠀⠀⣰⣿⡟⢠⣿⣙⡓⢦⡅⠀⠀⠀⠀⣀⡤⠤⢴⠋⣸⡟⡏⠀⠀⠀⠀⠀⠙⠁⠀⠀⠀⠘⣷⠤⣈⠑⠦⣄⠀⠀⠀⠀⠀⠀⣠⣶⣚⠉⠁⠀⢈⣉⢭⣷⡶⠔⠒⠒⠚⠛⠓⠀⠀                                                                                                  
⠀⣼⣿⠟⢀⣴⠋⢁⡖⠉⢀⡤⠖⡒⠉⢁⣄⣀⣾⠖⣇⣷⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⠀⠈⠑⠦⣀⠙⢦⡀⠀⣄⣠⡟⠳⣝⣦⠀⠀⠀⠙⠦⡈⠓⢤⡀⠀⠀⠀⠀⠀⠀                                                                                                  
⣼⣟⢉⣤⢸⣇⣀⣈⡀⠀⢀⣤⢰⣷⡤⣾⢻⣙⣟⣷⡟⠉⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⢿⠀⠀⠀⠀⠈⠳⠄⠘⢦⡀⢻⠁⠀⠀⠙⢷⡘⢦⣀⠀⠈⢦⡀⠙⢦⡀⠀⠀⠀⠀                                                                                                  
⣧⣿⣿⣧⡾⣏⢹⣿⣹⣿⣿⣇⢸⡿⣇⣿⡼⠻⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⠃⠈⡆⠀⠀⠀⠀⠀⠀⠘⢦⠹⣾⠀⠀⠀⠀⠀⠑⠀⠈⠉⠲⣼⣿⣦⣄⠹⣄⠀⠀⠀                                                                                                  
⠈⢿⣿⣟⣧⠘⢾⣮⢿⣿⢿⡟⢿⠟⠉⠛⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⠞⠹⡄⠀⡇⠀⢀⡤⠖⠒⠲⠤⣄⣳⣼⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠻⣿⣯⡙⢾⣆⠀⠀                                                                                                  
⠀⠀⣻⢯⠛⠅⢀⡄⠀⠀⠀⠀⠀⠀⠀⠀⢀⡤⠀⣀⣀⣀⠀⠀⠀⠀⠀⠀⣠⣞⡁⠀⠀⢇⠀⣇⡴⠋⠀⠀⠀⠀⠀⠀⠈⠻⣿⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢻⡻⣄⠉⢧⡀                                                                                                  
⠀⢠⡇⠀⠙⡶⠃⠀⢀⡖⠀⢰⠀⢠⢶⣴⡿⠚⠛⠻⢥⣉⠉⠓⠲⠦⠴⠚⠉⠀⠉⠙⢦⣸⠀⡿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠁⠈⢧⡀⠀                                                                                                  
⠀⣸⠀⢀⡠⢿⠀⠀⡼⣧⠀⡼⢷⡾⠀⠙⠃⠀⠀⠀⠀⠉⠳⣦⡀⠀⠀⠀⠀⠀⠀⠀⠀⣯⣸⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠳⡀                                                                                                  
⠀⣇⡴⠋⠀⢸⢀⡼⠁⠈⠻⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⢦⡀⠀⠀⠀⠀⠀⠀⣿⠇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⢳⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠹                                                                                                  
⠀⠋⠀⠀⠀⠘⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⢦⣄⠀⠀⠀⢰⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣛⢦⣤⣈⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀                                                                                                  
 __    __ _   _ _ __  _ __ ___ _   _  __ _  __ _  ____  __
 \\ \\/\/ /| | | || '__|  '_ ` _ \ | | |/ _` |/ _` ||_  / / _ \                                                                                                    
  \ /\ / | |_| || |   | | | | | || |_| (_| | (_| | / / |  __/                                                                                                      
   \/\/   \__, ||_|   |_| |_| |_| \__, |\__,_|\__,_|/___|\___|                                                                                                     
            |___/                     |___/                                                                                                                        
      [ pentest action graph generator ]                                                                                                                           
  output dir : test/
  actions    : 2

  [horizontal] svg : test/test_horizontal.svg
  [vertical] svg : test/test_vertical.svg
  [hybrid] svg : test/test_hybrid.svg
  [markdown]  md  : test/test.md

+-----+------------+--------+---------+
| #   | inputs     | action | results |
+-----+------------+--------+---------+
| 1   | 10.10.18.3 | nmap   | 80,9000 |
| 2   | 80,9000    | enum   | webapp  |
+-----+------------+--------+---------+
```
