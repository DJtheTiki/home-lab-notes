# Bandit Level [24 → 25]

## Objective
A daemon listening on port 30002 accepts the bandit24 password plus a 4-digit numeric PIN, and returns the bandit25 password if the PIN is correct. Since there's no way to know the PIN in advance, it has to be brute-forced — all 10,000 possible combinations (0000–9999).

## Commands used
- `nc` (netcat) — used to connect to and send data directly to a listening network service
- `seq -w` — generates a sequence of numbers with fixed width (leading zeros), so every PIN is exactly 4 digits
- `for` loop — repeats an action for every item in a list; here, once per possible PIN
- `|` (pipe) — passes the output of one command directly as input to another command, without needing to save it to a file first

## Approach
1. I first tested the service manually with `nc localhost 30002`, entering the bandit24 password plus a random PIN (`1234`). The service replied "Wrong! Please enter the correct current password and pincode. Try again." This confirmed the input format was correct — only the PIN was wrong.
2. Since checking 10,000 PINs by hand isn't realistic, I needed to automate it.
3. I learned that `seq -w 0000 9999` generates every number from 0000 to 9999 with leading zeros, which matches the required 4-digit PIN format.
4. I built a `for` loop that goes through every number in that sequence and prints a line combining the bandit24 password with the current PIN:
```
   for i in $(seq -w 0000 9999); do
       echo "PASSWORD $i"
   done
```
5. The task description mentioned that a new connection isn't needed
   for every attempt, which pointed me toward piping the entire loop's
   output into a single `nc` connection instead of reconnecting 10,000
   times:
```
   for i in $(seq -w 0000 9999); do
       echo "PASSWORD $i"
   done | nc localhost 30002
```
6. Running this sent all 10,000 password+PIN combinations through one
   connection.

   ## Challenges I ran into
I didn't run into a specific technical error, but I hadn't worked with a `for` loop combined with `nc` and a pipe before, so I needed guidance to understand how to structure the loop and connect it correctly to the netcat connection. Once it was explained step by step, it made sense — the loop generates all the input lines, and the pipe feeds them into the same connection one after another.

## What I learned
This level taught me how to combine several small building blocks | `seq`, a `for` loop, and a pipe into `nc` | to automate a brute-force attack over a single network connection. I now understand what a pipe does (passing one command's output directly into another command's input) and how a `for` loop can repeat an action across a large range of values without manual effort.
