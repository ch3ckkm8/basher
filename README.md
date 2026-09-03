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

If you don't provide a `[ColumnName]`, the table just uses the variable name itself.

Variables that share the same `[ColumnName]` are grouped into that one column, stacked as separate rows.

```bash
source ./basher foo bar BADCOL
# Error: column name must be wrapped in brackets, e.g. [ColumnName]
```

### Exclude a variable from the table entirely
Pass `[notable]` instead of a column name to save and export the variable as normal, but keep it out of `--table`/`--per-index` (and the generated markdown file) completely. It still shows up in `--list` and `--load`.

```bash
source ./basher session_id 8f3a1c2e [notable]
# Set & exported: session_id = 8f3a1c2e (excluded from --table)
```

### Get a variable
```bash
source ./basher target
# → 10.10.10.6
```

### List all stored variables
Also exports every stored variable into the current shell as a side effect — handy for refreshing a fresh terminal without a separate `--load`.

```bash
source ./basher --list
# VARIABLE             VALUE                COLUMN
# --------             -----                ------
# target               10.10.10.6           IP
# user                 myuser               (default)
# session_id           8f3a1c2e             (excluded from --table)
```

### Display all variables as a table
Prints all stored variables as a table and also exports every one of them into the current shell (same as `--list`) — so running `--table` alone is enough to pick up variables saved in an earlier session.

**Columns** come from each variable's custom `[ColumnName]` (or its own varname, if none was set). Variables sharing a `[ColumnName]` stack as separate rows in that one column.

**Rows** are grouped and sorted by the *primary* numeric index in the variable name — e.g. `user1`, `pass1`, `target1`, and `port1` all land on row "1" together, `user2`/`pass2` land on row "2", and so on, sorted with the lowest index on top. A trailing `_N` suffix is treated as a **sub-index** of the number before it, not a separate row — so `creds1_1` and `creds1_2` both count as index `1`, right alongside `user1`. Variables with no index at all (like a plain `notes`) are placed together on the final row. Shorter columns are simply left blank for rows that don't have an entry.

If several variables end up sharing **both** the same index **and** the same `[ColumnName]` (e.g. `loot5`, `loot5_1`, and `loot5_2` all tagged `[Loot]`), they'd otherwise collide into one table cell — instead of losing data, each one gets its own row under that column, in the order they were set. Other columns on that index (like `target5` below) just show their value on the first of those rows and stay blank on the rest.

```bash
source ./basher target5 10.10.10.5
source ./basher loot5 "gold coins" [Loot]
source ./basher loot5_1 "silver coins" [Loot]
source ./basher loot5_2 "bronze coins" [Loot]

source ./basher --table 5
# Index: 5
# Loot         target5
# ------------ ----------
# gold coins   10.10.10.5
# silver coins
# bronze coins
```

```bash
source ./basher user1 b0b
source ./basher host1 desktop-p2d21
source ./basher creds1 b0b:p4ss [Credentials]
source ./basher creds1_2 alt-cred:p4ss2 [AltCreds]
source ./basher user2 alice

source ./basher --table
# user1 host1         Credentials AltCreds
# ----- ------------- ----------- --------------
# b0b   desktop-p2d21 b0b:p4ss    alt-cred:p4ss2
#       (row for user2 below)
```

This also automatically writes the same table as a Markdown file — `basher_table.md` — saved in the same directory as the `basher` script itself. It's overwritten every time you run `--table` (with no index argument).


| user1 | host1 | Credentials | AltCreds |
| --- | --- | --- | --- |
| b0b | desktop-p2d21 | b0b:p4ss | alt-cred:p4ss2 |


### Show only one index's variables
Pass a numeric index as a second argument to `--table` to print just the variables grouped under that index (again including any `_N` sub-indexed ones). This is a quick lookup and doesn't touch `basher_table.md`.

```bash
source ./basher --table 1
# Index: 1
# user1 host1         Credentials AltCreds
# ----- ------------- ----------- --------------
# b0b   desktop-p2d21 b0b:p4ss    alt-cred:p4ss2

source ./basher --table 99
# No variables found with index: 99
```

### Display one mini-table per index
`--per-index` walks through every index you have (lowest to highest, with any no-index variables last) and prints a separate mini-table for each — handy when you're juggling several hosts/targets and want them broken out individually instead of one wide combined table. It also exports every variable and (over)writes `basher_table.md` with all the sections, one per index.

```bash
source ./basher --per-index
# user1 host1         Credentials AltCreds
# ----- ------------- ----------- --------------
# b0b   desktop-p2d21 b0b:p4ss    alt-cred:p4ss2
#
# Index: 2
# user2 Credentials
# ----- ------------
# alice alice:secret
#
# Index: No index
# notes
# -------------------
# flat note, no index
```

```md
### Index: 1

| user1 | host1 | Credentials | AltCreds |
| --- | --- | --- | --- |
| b0b | desktop-p2d21 | b0b:p4ss | alt-cred:p4ss2 |

### Index: 2

| user2 | Credentials |
| --- | --- |
| alice | alice:secret |

### Index: No index

| notes |
| --- |
| flat note, no index |
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

Variables are stored in `~/.basher_store`, one per line, as `KEY=VALUE` followed by a custom column name and a `[notable]` flag, both optional, separated by the Unit Separator character (`\x1f`, not a visible character in a normal text editor):

```
target=10.10.10.6␟IP␟
user=myuser␟␟
session_id=8f3a1c2e␟␟1
```

You don't need to edit this file by hand — it's managed for you by the `set`, `--delete`, and `--clear` commands.

> **Note:** older versions of this script used a literal tab as the separator, which had a subtle bug (bash collapses consecutive tabs, corrupting entries that had no custom column). If `basher` finds any leftover tab-delimited entries in your store file, it automatically migrates them to the current format the next time you run it — your variable values are preserved, but any custom `[ColumnName]`/`[notable]` tags on those specific entries are reset to default, since they can't be reliably recovered from the old corrupted format. You'll see a one-time message like:
> ```
> Migrated 3 variable(s) from an older basher storage format.
> Values were preserved; any custom [ColumnName]/[notable] tags on those were reset — feel free to re-set them.
> ```

You can override the storage location with the `BASHER_STORE` environment variable:

```bash
export BASHER_STORE=/tmp/my_custom_store
source ./basher target 10.10.10.6
```

---

## Real-world example

```bash
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

> source basher --table 1
Index: 1
IP         Hostname  Notes            Ports   Credentials
---------- --------- ---------------- ------- ------------
10.10.18.3 linuxhost bob-sworkstation 80,9000 b0b:p4ssw0rd
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
⠀⠀⠀⠀⠀⠀⠀⢠⡖⠋⠀⣠⠖⠋⠁⠀⠀⠀⢀⡀⠀⠀⠀⠀⢀⣀⠤⠞⠁⠀⠉⠀⠀⠀⠀⠀⢀⣀⣀⠀⠉⠙⠒⠦⢴⣋⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣈⣭⠽⠿⠟⠓⠀⠀⠀                                                                                                  
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

---

## Variable naming rules

Variable names must start with a letter or underscore, and contain only letters, digits, and underscores.

```bash
source ./basher my_var value      # ✅ valid
source ./basher 1var value        # ❌ invalid
source ./basher my-var value      # ❌ invalid
```

Custom table column names (the optional `[ColumnName]` argument) must be wrapped in square brackets, but otherwise have no character restrictions. `[notable]` is reserved and excludes the variable from `--table`/`--per-index` instead of being treated as a literal column name.
