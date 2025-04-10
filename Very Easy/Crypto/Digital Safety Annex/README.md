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
```bash
Welcome to the Digital Safety Annex!
We will keep your data safe so you don't have to worry!

[0] Create Account 
[1] Store Secret
[2] Verify Secret
[3] Download Secret
[4] Developer Note
[5] Exit

[+] Option > 4

We are here to prove that DSA is waaayyyy better than El Gamal!
We also modified our signature algorithm to use the super secure SHA-256. No way you can bypass our authentication. If you must try, be sure to bring tissues for your tears of failure.
I'll throw you a bone, these are public record anyway:

p = 20456200795023025692958927027179968362406134218020382295746979677488535559846352299024373764766692133594352864793563272558946919303383407870276716291012017330004421918846922824017183380147559598222249403822492341460107900774534613914613180158171832676561697320727293036476329750065474208935491777777692598232959965945001550943219388580471749979773626932151144539971150486334503639383423377723317044824174113098484505983403469118370190860175762916520692683003134217151459299174357461553572701841532189840675438451080743779278146546400067750596862380347768688256139255207888127411703376012366568872793616789466612032337
q = 13923371277455179830687400981554961612548725809674329440204610344879
g = 15945028550811775298748908503927022305478095700759947506016051952611383024828617073778944776312909383689936118562899208134940337529868932092672372894468794479992545139308691928837574462132286548022542660274071750089264838534025060495703747844377931023638219448413071701834040249313695104579458030379308433354292389979733654496415329799558204607062794671404587749255373869849934604188571200177451976745362839899116820239189295167114818542776044171369151117485794686507810838777361324997668224043677320794369200804626027492567390060655272022705939350205152529210038615806521991969087127961036146821874401003731714960947
[+] Test user log (y/n):
```
keep in mind the "bone", that throws us the devs and continue:
```bash
[+] Test user log (y/n): y
Enter your password : 5up3r_53cur3_P45sw0r6

[((3377387044605854864244776383838507013440980236437974998098072382996, 11387833179588560538843749860034514565075597428412591740442656240480), 'a0aad39c9280260016dabaed8ca0c16d812ed8f2ccaa79eed07908f6bc74fb48'), ((7906786913496186064066465588197598444235427884230336650575526109352, 1847756693147672458062177923750270639690761887287075186402368441722), 'b61c744b656adfca5049503c898073fefce49413e072505541e78460c02345ac'), ((3213164992442186303528464302147186878090756582351934442608005651412, 12663418635463918006131765639346514857655432447165399696265693870328), '625e8f4530e14927bd3095b35917e127a056517ba11fa7570deae5485a1a8503'), ((339240333272558047160230761420199199162538960401185104869142414030, 5426835505877592751489592345720715076591870301295365183149507787886), '0e9722da720ecebce84ce77efa3f047e7a9b1c1fb11264be7d5cc1adbb8a73a5'), ((7786306799360191313653456864206512635736085135571911758804132769277, 1316460418709065836887345405266944465638081091100703446448789048792), '36f6e72c03df0167409761fa929d4574b13a7d513a903a8c49193836c0cb34cd'), ((389872743388110861771370092060917559182002000956559079666102844760, 5462903427368158105284013242821530740680377538506609089366407682579), 'e20824d197269d9df0949caf37c7df2676ffc9e6b26bf3fd6e34bcb872651445')]
```
Eureka!, we found the stored messages. we are on track. the last thing to do is to find the steps that get throught to output this.

## Explotation
knowing all these information the only way to retraive the messages is using the option 3 in he menu. but we required some information about the mesage
before:
* `k` value
* `x` value
* `request id` of the message
