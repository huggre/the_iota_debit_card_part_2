# Integrating physical devices with IOTA — The IOTA debit card, Part 2

The 7th part in a series of beginner tutorials on integrating physical devices with the IOTA protocol.

![img](https://miro.medium.com/max/700/1*k7Js1ZqZpAooZ9X6m5x7rw.jpeg)

------

## Introduction

This is the 7th part in a series of beginner tutorials where we explore integrating physical devices with the IOTA protocol. This tutorial is the second part in a sequence of tutorials where we will try to replicate a traditional fiat based debit card payment solution with an IOTA based solution. In this second tutorial we will build a simple LED circuit representing the physical device that will allow us to pay for its service using our new IOTA debit card.

------

## The Use Case

Now that we have our new IOTA debit card configured and ready to go as described in the [previous tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-the-iota-debit-card-part-1-42dc1a05f18), we will shift our attention to how we can use the card to pay for services. In the [previous tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-the-iota-debit-card-part-1-42dc1a05f18) i used the coffee maker in the reception and the swimming pool locker as examples where we could integrate our new IOTA debit payment solution. But to be honest, a payment solution like this could probably be integrated into any machine or device that accept payments for its services, such as vending machines, slot machines, parking meters or the hotel room refrigerator from the [first tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb). And why stop there? The guest could even use his IOTA debit card to pay for dinner in the hotel restaurant or items the hotel gift shop, preventing the hotel owner from being charged additional fees by Visa or any other third party payment provider. The possibilities are basically endless.

We will use a simplified version of the LED powered circuit from the [first tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb) to represent the device being payed in this tutorial. The main objective of this tutorial is to send valued transactions using our IOTA debit card so we will remove any unneeded components, wiring and coding from the [first tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb) as this is not needed to get the point across.

------

## About value transactions

A value transaction is an IOTA transaction that transfer IOTA tokens between IOTA addresses, as appose to [tutorial 5](https://medium.com/coinmonks/integrating-physical-devices-with-iota-using-rfid-with-iota-868c15e0a040) where we sent non-value transactions to the tangle. A typical value transaction actually consist multiple transactions packaged in a *bundle*. A *bundle* consist of 3 types of transactions:

1. Input TXs
   Negative value transaction(s) that deduct value from the sender address(es)
2. Output TXs
   Positive value transaction(s) that add value to the receiver address
3. Unspent TXs
   Zero sum transaction(s) that transfer remaining value from the sender address(es) to a new sender address. The purpose of these transactions is to prevent future spending from an already used sender address.

While this sounds rather complicated, the good news is that we don’t have to worry about the complexity of setting up the bundle, this will all be taken care of by the PyOTA library.

*Note!*
*Notice that when all transactions in the bundle have been executed, the total amount of existing IOTA’s is unchanged, only the ownership addresses of the IOTA tokens has changed. This is of course an essential property of the IOTA protocol that must be enforced at all times.*

------

## Wiring the project

Next, lets have a look at wiring up the project. As described earlier, our circuit for this tutorial will be a simplified version of the circuit used in the [first tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb). I will remove the relay and battery from the circuit as they are not needed to demonstrate the point we are trying to make in this tutorial. If you like to include them in your version of the project, that’s OK. The Python code will work fine with or without them.

See [tutorial 5](https://medium.com/coinmonks/integrating-physical-devices-with-iota-using-rfid-with-iota-868c15e0a040) for information on how to set up the MFRC522 RFID reader/writer if you haven’t done so already.

See [tutorial 1](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb) for information on hooking up the LED. If you want to simplify the LED circuit as i did when writing this tutorial, you can skip the relay and battery and connect the LED directly to the Raspberry PI GIO pins. Note that you still need the resistor in the circuit or you may overload and damage the LED.

![img](https://miro.medium.com/max/700/1*fPYX6gMhhf_GO9AaZwN64g.png)

*Note!*
*As pin 6 that was used for GROUND in the first tutorial now is occupied by the the MFRC522, i chose to use pin 9 instead as GROUND for my LED. In case you’r using a breadboard, you can also connect ground from both the MFRC522 and the LED to a common ground on the breadboard.*

![img](https://miro.medium.com/max/367/1*LyZnkPjYofggUfFNcGhzlg.png)

------

## The Python code

The main objective of this tutorial is to use our IOTA debit card to pay a physical device for its service. In my example the service consist of a blinking LED. Before we start coding, lets to a quick rundown of the logic behind the code and how it works.

1. The python script starts by checking the balance of the receiver (hotel owner) address so that we have a baseline for monitoring when new payments are being added to the address.
2. Next, the python script shows a welcome message where you will be asked how many “blinks” you want to purchase. In my example i have set set the price to 1 IOTA for 1 blink.
3. Next, you will be asked to hold your IOTA debit card close to the RFID reader.
   As soon as the RFID reader detects the card, the Python script will read the IOTA seed from the card.
4. Next, we check if there is enough funds on the debit card seed to pay for the number of blinks your ordered.
5. If funding is OK, we define a new transaction before sending it to the Tangle.
6. Next, we start monitoring the receiver (hotel owner) address balance to see if any new funds are being added. Meaning, if the transaction from the previous step has been confirmed.
7. Finally, when the transaction has been confirmed we blink the LED the number of times ordered.

*Note!
If you followed the previous tutorials in this series you will be aware of the address reuse problem that will occur when the hotel owner wants to spend any funds from the receiver address where the LED blink tokens are collected. In* [*tutorial 4*](https://medium.com/coinmonks/integrating-physical-devices-with-iota-price-and-addresses-4f352e321cbb) *we solved the address reuse problem using an LCD display with a QR code as a convenient way of enabling the user to pay using his mobile IOTA wallet. Now that we use our IOTA debit card to do the payment, there is no need for the user to know what receiving address is being used to pay the LED, this can all be managed behind the scenes. If you want to implement a solution for the address reuse problem in this project, use a similar approach as we did in* [*tutorial 4*](https://medium.com/coinmonks/integrating-physical-devices-with-iota-price-and-addresses-4f352e321cbb)*, without the LCD.

```python
# Imports some required libraries
import iota
from iota import Address
import RPi.GPIO as GPIO
import MFRC522
import signal
import time

# Setup O/I PIN's
LEDPIN=12
GPIO.setmode(GPIO.BOARD)
GPIO.setwarnings(False)
GPIO.setup(LEDPIN,GPIO.OUT)
GPIO.output(LEDPIN,GPIO.LOW)

# URL to IOTA fullnode used when interacting with the Tangle
iotaNode = "https://nodes.thetangle.org:443"
api = iota.Iota(iotaNode, "")

# Hotel owner recieving address, replace with your own recieving address
hotel_address = b'NYZBHOVSMDWWABXSACAJTTWJOQRPVVAWLBSFQVSJSWWBJJLLSQKNZFC9XCRPQSVFQZPBJCJRANNPVMMEZQJRQSVVGZ'

# Some variables to control program flow 
continue_reading = True
transaction_confirmed = False
       
# Capture SIGINT for cleanup when the script is aborted
def end_read(signal,frame):
    global continue_reading
    print "Ctrl+C captured, ending read."
    continue_reading = False
    GPIO.cleanup()

# Function that reads the seed stored on the IOTA debit card
def read_seed():
    
    seed = ""
    seed = seed + read_block(8)
    seed = seed + read_block(9)
    seed = seed + read_block(10)
    seed = seed + read_block(12)
    seed = seed + read_block(13)
    seed = seed + read_block(14)
    
    # Return the first 81 characters of the retrieved seed
    return seed[0:81]

# Function to read single block from RFID tag
def read_block(blockID):

    status = MIFAREReader.MFRC522_Auth(MIFAREReader.PICC_AUTHENT1A, blockID, key, uid)
    
    if status == MIFAREReader.MI_OK:
        
        str_data = ""
        int_data=(MIFAREReader.MFRC522_Read(blockID))
        for number in int_data:
            str_data = str_data + chr(number)
        return str_data
        
    else:
        print "Authentication error"

# Function for checking address balance 
def checkbalance(hotel_address):
    
    address = Address(hotel_address)
    gb_result = api.get_balances([address])
    balance = gb_result['balances']
    return (balance[0])

# Function that blinks the LED
def blinkLED(blinks):
    for x in range(blinks):
        print("LED ON")
        GPIO.output(LEDPIN,GPIO.HIGH)
        time.sleep(1)
        print("LED OFF")
        GPIO.output(LEDPIN,GPIO.LOW)
        time.sleep(1)

# Get hotel owner address balance at startup
print("\n Getting startup balance... Please wait...")
currentbalance = checkbalance(hotel_address)
lastbalance = currentbalance

# Hook the SIGINT
signal.signal(signal.SIGINT, end_read)

# Create an object of the class MFRC522
MIFAREReader = MFRC522.MFRC522()

# Show welcome message
print("\nWelcome to the LED blink service")
print("\nThe price of the service is 1 IOTA for 1 blink")
blinks=input("\nHow many blinks would you like to purchase? :")
print("\nHold your IOTA debit card close to ")
print("the reader to pay for the service...")
print("Press Ctrl+C to exit...")

# This loop keeps checking for near by RFID tags. If one is found it will get the UID and authenticate
while continue_reading:
           
    # Scan for cards    
    (status,TagType) = MIFAREReader.MFRC522_Request(MIFAREReader.PICC_REQIDL)

    # If a card is found
    if status == MIFAREReader.MI_OK:
        print "Card detected"
    
    # Get the UID of the card
    (status,uid) = MIFAREReader.MFRC522_Anticoll()

    # If we have the UID, continue
    if status == MIFAREReader.MI_OK:

        # Print UID
        print "Card read UID: %s,%s,%s,%s" % (uid[0], uid[1], uid[2], uid[3])
    
        # This is the default key for authentication
        key = [0xFF,0xFF,0xFF,0xFF,0xFF,0xFF]
        
        # Select the scanned tag
        MIFAREReader.MFRC522_SelectTag(uid)
        
        # Get seed from IOTA debit card
        SeedSender=read_seed()
        
        # Stop reading/writing to RFID tag
        MIFAREReader.MFRC522_StopCrypto1()
                      
        # Create PyOTA object using seed from IOTA debit card
        api = iota.Iota(iotaNode, seed=SeedSender)
        
        print("\nChecking for available funds... Please wait...")
               
        # Get available funds from IOTA debit card seed
        card_balance = api.get_account_data(start=0, stop=None)
            
        balance = card_balance['balance']
        
        # Check if enough funds to pay for service
        if balance < blinks:
            print("Not enough funds available on IOTA debit card")
            exit()
        
        # Create new transaction
        tx1 = iota.ProposedTransaction( address = iota.Address(hotel_address), message = None, tag = iota.Tag(b'HOTEL9IOTA'), value = blinks)

        # Send transaction to tangle
        print("\nSending transaction... Please wait...")
        SentBundle = api.send_transfer(depth=3,transfers=[tx1], inputs=None, change_address=None, min_weight_magnitude=14, security_level=2)
                       
        # Display transaction sent confirmation message
        print("\nTransaction sent... Please wait while transaction is confirmed...")
        
        # Loop executes every 10 seconds to checks if transaction is confirmed
        while transaction_confirmed == False:
            print("\nChecking balance to see if transaction is confirmed...")
            currentbalance = checkbalance(hotel_address)
            if currentbalance > lastbalance:
                print("\nTransaction is confirmed")
                blinkLED(blinks)
                transaction_confirmed = True
                continue_reading = False
            time.sleep(10)
```

You can download the source code from [here](https://gist.github.com/huggre/71c6aadb4e4e3d37018dc3acee5a17a0)

## Running the project

To run the the project, you first need to save the code in the previous section as a text file in the same folder as where you installed the MFRC522-python library.

Notice that Python program files uses the .py extension, so let’s save the file as **iota_debit_card_pay.py** on the Raspberry PI.

To execute the program, simply start a new terminal window, navigate to the folder where you saved *iota_debit_card_pay.py* and type:

**python iota_debit_card_pay.py**

You should now see the code being executed in your terminal window asking you for the number of blinks you would like to purchase.

------

## Improvement to the previous tutorial

Now that we have a basic understanding of how to create value transactions with PyOTA, it should be pretty straight forward to add a new option to the [previous tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-the-iota-debit-card-part-1-42dc1a05f18) that will allow us to transfer funds from the hotel owner seed to the debit card seed. As you may remember, we had the function of generating a new address that was to be used when transferring new funds to the IOTA debit card. Now its just a matter of reversing the payment process used in this tutorial. Instead of sending value transactions from the IOTA debit card seed to the hotel owner address, we instead send value transactions from the hotel owner seed to the IOTA debit card address. Implementing this feature in the code from the [previous tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-the-iota-debit-card-part-1-42dc1a05f18) should be a nice learning exercise that i will leave up to you as the reader.

------

## What's next?

You have probably noticed that you are not asked for any authorization or credentials when paying with your IOTA debit card. While this might be fine in some use cases, there may be other use cases where this is not acceptable. Imagine if you lost your IOTA debit card and it was picked up by a bad actor. Without any protection mechanism there would be nothing preventing him from using your card. In the next tutorial we will be addressing this issue by implementing a PIN code authorization mechanism for our IOTA debit card. Stay tuned…

------

## Donations

If you like this tutorial and want me to continue making others, feel free to make a small donation to the IOTA address shown below.

![img](https://miro.medium.com/max/382/1*j2ENIzmDzXcGSgAdY4w-Jw.png)

NYZBHOVSMDWWABXSACAJTTWJOQRPVVAWLBSFQVSJSWWBJJLLSQKNZFC9XCRPQSVFQZPBJCJRANNPVMMEZQJRQSVVGZ

