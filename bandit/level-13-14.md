# Bandit Level [13 → 14]

## Objective
I needed to log into the next level using an SSH private key instead of a password.


## Background: what I learned about SSH keys

Before solving this level, I didn't fully understand how SSH authentication
works. Through this challenge, I learned that SSH can use a key pair instead
of a password:

  - A **private key** stays only on your own machine and must never be shared —
  it proves your identity.
  - A **public key** can be placed on the server you want to connect to — it's
  used to verify that whoever holds the matching private key is allowed in.
  - When you connect, the server and your machine use these two keys together
  to create a secure, encrypted connection, without ever sending your actual
  password over the network.


## Commands used
- `nano` — a simple command-line text editor; I used it to paste and save the copied private key into a local file
- `cat` — it displays/outputs the content of a file, in my situation the SSH key
- `chmod` — restricts the key file's permissions so only the
  owner can read/write it (SSH refuses to use a key with overly open
  permissions)
- `ssh -i [keyfile] user@host -p [port]` — connects to a server or other client using a specific private
  key instead of a password


## Approach
1. I tried to log in to bandit14 using the password I had from the previous level, but the login was rejected (password authentication was disabled for this level).
2. After this, I tried to print the key with `cat sshkey.private`
3. Then I copied the SSH key, and I used `nano sshkey.private` to write down the key, I saved it with Ctrl + O, then Enter, then Ctrl + X.
4. I used `chmod 600` to restrict the file's permissions.
5. And then I could use `ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220`.
6. I successfully connected to the next level using the private key.

## Challenges I ran into
At first SSH rejected the key with a 'UNPROTECTED PRIVATE KEY FILE' warning, which taught me that SSH enforces strict file permission rules on private keys.

## What I learned
This level gave me a solid practical understanding of asymmetric cryptography in the context of authentication, something I'd only known in theory before. I now understand why SSH key-based login is considered more secure than password-based login, and I can explain the difference between a private and public key confidently.

Screenshots of the process:

<img width="723" height="409" alt="20d2b952-0207-4659-a92c-7fa3669c6e2e" src="https://github.com/user-attachments/assets/59790b7e-1ef8-4793-aa6e-d5fc40cb550e" />
