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
`2`, verify the signature belongs to a supouse message or a stored message
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
