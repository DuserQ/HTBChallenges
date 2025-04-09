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


