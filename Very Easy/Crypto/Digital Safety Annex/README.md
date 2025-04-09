![Alt text](https://resources.hackthebox.com/hubfs/HacktheBox%20Logo.png)

## Challenge Description
Here at D.S.A we store all your super secret information in a secure vault until you provide us with proof you are who you say you are. We even use SHA256 instead of the weak SHA1! We are so confident, we invite all who wish to show us otherwise!



## Enumeration
In this challenge we're provided with a protected zip folder(which password is below the download buttom as you can see in the image) 

![Alt text](https://github.com/user-attachments/assets/2ce8e77e-3b98-4fff-9323-ce947876be45)

that contains 4 files:
* `server.py`: the script that runs when whe connect to the instace
* `_account.py`: show how the user are created
* `_annex`: source class for all the processes in the server 
* `_dsa.py`: provides the digital signatures for the messages that you store in the program

## Source Code Analysis
the program starting stored some messages 
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

``
