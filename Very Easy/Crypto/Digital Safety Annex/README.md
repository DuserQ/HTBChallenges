![Alt text](https://resources.hackthebox.com/hubfs/HacktheBox%20Logo.png)

## Challenge Description
Here at D.S.A we store all your super secret information in a secure vault until you provide us with proof you are who you say you are. We even use SHA256 instead of the weak SHA1! We are so confident, we invite all who wish to show us otherwise!


## Enumeration
In this challenge we're provided with a protected zip folder(which password is below the download buttom as you can see in the image) 

![Alt text](https://github.com/user-attachments/assets/2ce8e77e-3b98-4fff-9323-ce947876be45)

that contains 4 files:
* `server.py`: the script that runs when whe connect to the instace
* `_account.py`: show how the user are created
* `_annex.py`: source class for all the processes in the server 
* `_dsa.py`: provides the digital signatures for the messages that you store in the program

## Source Code Analysis
the program starting calling the class `Annex()` (that is store in the `_annex.py` file)  reassigned to a var called `annex` and stored some messages 
```python
def main():
    annex = Annex()
    annex.create_account("Admin", "5up3r_53cur3_P45sw0r6")
    annex.create_account("ElGamalSux", HTB_PASSWD)
    annex.sign("ElGamalSux", "DSA is a way better algorithm", HTB_PASSWD)
    annex.sign("Admin", "Testing signing feature", "5up3r_53cur3_P45sw0r6")
    annex.sign("ElGamalSux", "I doubt anyone could beat it", HTB_PASSWD)
    annex.sign("Admin", "I should display the user log and make sure its working", "5up3r_53cur3_P45sw0r6")
    annex.sign("ElGamalSux", "To prove it, I'm going to upload my most precious item! No one but me will be able to get it!", HTB_PASSWD)
    annex.sign("ElGamalSux", FLAG, HTB_PASSWD)
```
then starting printing the menu

```python
def show_menu():
    return input("""
Welcome to the Digital Safety Annex!
We will keep your data safe so you don't have to worry!

[0] Create Account 
[1] Store Secret
[2] Verify Secret
[3] Download Secret
[4] Developer Note
[5] Exit

[+] Option > """)
```
we see that the numbers from 0 to 5 are the range of functions that have the program that are to `user_inp` like is show below
```python
while True:
        user_inp = show_menu()
```
option `0` is to create a user
```python
if user_inp == '0':
            account_username = annex.create_account()
```
option `1` for store a message and then is sign by the `annex` class, later the message's signature generated is later display in screen
```python
 elif user_inp == '1':
            if not account_username:
                print("\n[!] You need to create an account with the Annex first before you can store any secrets!")
            else:
                while True:
                    message = input("\nPlease enter you super secret message: ")
                    r, s = annex.sign(account_username, message)

                    if r > 0 and s > 0:
                        break

                print(f"\n[+] Here is your signature (r, s): ({r}, {s})")
                print("Keep your signature safe!!")
```
`2`, verify the signature belongs to a supposed message or a stored message
```python
elif user_inp == '2':
            signature = input("\nPlease enter the signature (r,s) separated by a comma: ")
            signature = re.search(r'^(\d+),(\d+)$', signature)

            if not signature:
                print("\n[!] Sorry, need a valid signature to verify message!")
                continue
            
            message = input("\nPlease enter the message you wish to verify: ").strip()
            if not message:
                print("\n[!] Sorry, need a valid message to continue verification!")
                continue

            signature = (int(signature.group(1)), int(signature.group(2)))

            if annex.verify(message, signature):
                print("[+] Message has been successfully verified!")
            else:
                print("[!] Message could not be verified! Please make sure your signature and messages are correct!")
```
`3`, in base on the username registered you can download the secret messages that you stored in the program
```python
 elif user_inp == '3':
            uname = input("\nPlease enter the username that stored the message: ")
            if not uname in annex.vault:
                print("\n[!] Sorry, need valid existing username to download secret!")
                continue

            req_id = input("\nPlease enter the message's request id: ")
            if not req_id.isdigit() or not (0 <= int(req_id) < len(annex.vault[uname])):
                print("\n[!] Sorry, need valid request id to download secret!")
                continue
            
            req_id = int(req_id)
            if uname == account_username:
                account = annex.users[account_username]

                if account.login():
                    h, msg, sig = annex.vault[uname][req_id]
                    print(f"\n[+] Here is your message: {msg}")
                else:
                    print("[-] Invalid username and/or password!")
            else:
                k = input_number("\nPlease enter the message's nonce value: ")
                if not k:
                    print("\n[!] Sorry, need a valid nonce to download secret!")
                    continue
            
                x = input_number("\n[+] Please enter the private key: ")
                if not x:
                    print("\n[!] Sorry, need a valid private key to download secret!")
                    continue

                annex.download(x, k, req_id, uname)
```
`4`, a some notes that the developers leave for the people trying to break his system (not humble for them)
```python
 elif user_inp == '4':
            print("\nWe are here to prove that DSA is waaayyyy better than El Gamal!\nWe also modified our signature algorithm to use the super secure SHA-256. No way you can bypass our authentication. If you must try, be sure to bring tissues for your tears of failure.\nI'll throw you a bone, these are public record anyway:\n")

            p, q, g = annex.dsa.get_public_params()

            print(f'{p = }')
            print(f'{q = }')
            print(f'{g = }')

            inp = input("[+] Test user log (y/n): ").lower()
            if inp == 'y':
                if annex.users['Admin'].login():
                    print(f'\n{annex.user_log}')
        
        elif user_inp == '5':
            print("[!] Leaving the Annex. Thanks for choosing DSA!")
            break
        
        else:
            print("[!] Invalid option.")
```
## Solution
lets start connecting to the instance
```bash
$ nc 94.237.63.16 34051

Welcome to the Digital Safety Annex!
We will keep your data safe so you don't have to worry!

[0] Create Account 
[1] Store Secret
[2] Verify Secret
[3] Download Secret
[4] Developer Note
[5] Exit

[+] Option > 
```
as we know, some messages were store as thirst parameters of the program. between them the flag's one:
```python
annex.sign("ElGamalSux", FLAG, HTB_PASSWD)
```
so, we need find a way to retraive this flag, if you can see there's a calling to the class `Annex`, specifically the function `sign`.
that give us some clues about how to solve, lets take a look to this function in the `_annex.py` file.
```python
def sign(self, username, message, password=""):
        account = self.users[username]
        
        if not account.login(password):
            print("[!] Invalid Password!\n")
            return (0, 0)

        msg = message.encode()
        h = sha256(msg).hexdigest()
        
        r, s = self.dsa.sign(h, account.k_max)
        
        self.log_info(account, msg, h, (r, s))
        
        return (r, s)
```
all the messages that was sending are storing in the function `log_info`
```python
    def log_info(self, account, msg, h, sig):
        _id = account.stored_msgs
        if account.username not in self.vault:
            self.vault[account.username] = []
        
        self.vault[account.username].append((h, msg, (str(sig[0]), str(sig[1]))))
        self.user_log.append((sig, h))
        account.stored_msgs += 1
```
Bingo!, we can see the structured where are all saved. but we still have a problem and its to see all this encoded messages.
but if we had take notice about the option menu in `server.py`, the 4rd option has a peculiar syntax, and its after the notes of the devs:
```python
 inp = input("[+] Test user log (y/n): ").lower()
            if inp == 'y':
                if annex.users['Admin'].login():
                    print(f'\n{annex.user_log}')
```
knowing this we need to know what print when this condition happens, let get back to the program:


