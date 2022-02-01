# Snort-Practice
Getting Snort IDS running and practicing using &amp; evading it

Snort is a very popular open source IDS (intrusion detection system).  As such, it is a good candidate to do some practicing with. In this document, I'll go through the steps I took to get Snort set up on a Kali Linux VM, set some rules for it, and practice using those rules, as well as exploring how an attacker might try to subvert Snort (or other IDS's).

--- Getting Set Up ---

Kali's default sources list doesn't include access to Snort.  Why? Good question. My thinking is that Kali is primarily a distro for OFFENSIVE security, and setting up an IDS is outside of that mission.  In fact, an IDS probably ins't meant to be on a workstation at all.  Regardless, getting Snort running requires first adding to Kali's default repository.

First, open the sources list to edit:
```
sudo nano /etc/apt/sources.list
```

Second, add this to the end of the document:
```
deb http://deb.debian.org/debian buster main
```

It should look like this:
![Screen Shot 2022-01-31 at 10 06 48 PM](https://user-images.githubusercontent.com/91093176/151915644-c628e3d5-044a-4f4c-9d2a-c44ee2c886f7.png)

Next, update your list of sources:
```
sudo apt update -y
```

Finally, install Snort:
```
sudo apt install snort
```
NOTE: You'll need to choose your home network (using CIDR notation) when you set it up, but you can also skip it and add that info in later.

And now Snort is installed on your Kali box!


--- Testing Rules ---

Now, to get started testing some quick & dirty Snort rules, I created a simple rules file in my home directory:
```
touch test.rules
```

To test that I had Snort set up correctly (at least so far), I ran it in test mode:
```
sudo snort -T -i eth1 -c ./test.rules
```

If all is well, it will let you know.

![Screen Shot 2022-01-31 at 10 15 40 PM](https://user-images.githubusercontent.com/91093176/151916299-d09ff6c5-bfdd-41d5-99a3-7704c6fb279c.png)


The first and most basic rule to write and test is alerting for pings.  Snort rules need to follow a specific structure that looks like this:

![snort-syntax](https://user-images.githubusercontent.com/91093176/151916459-5948f0fb-edff-4856-9418-ce94ab4e1657.png)

So my rule for ping scans ended up looking like this:

```
alert icmp any any -> 192.168.56.0/24 any (msg: ”ping attempt”; sid:1000001;)
```

After saving and exiting the rules file, I ran Snort in IDS mode, spitting the output to the console rather than a log:

```
sudo snort -A console -q -c ./test.rules -i eth1
```

Then, after going to another VM and pinging my Kali box, I get the alert!  On my first try writing a rule! 

![Screen Shot 2022-01-31 at 10 22 13 PM](https://user-images.githubusercontent.com/91093176/151916857-384e683d-03f0-4765-933f-93f7a3ee3c34.png)

--- Scanning for NMAP Scans ---

The next step would be scanning for behaviour that indicates an attacker is enumerating my network, right?  So, the logical step is to scan for NMAP scans.  At first, my mind was going to complex rules looking for a whole range of scans (which I would have no idea how to write), but thanks to a little research (https://www.hackingarticles.in/detect-nmap-scan-using-snort/), I realized it's much simpler than that. I don't need to identify someone scanning a huge range, I just need to identify one piece of traffic that is part of NMAP scans, but doesn't show up otherwise.  In this case, someone connecting to port 22 (I don't have any SSH set up that would need connections to port 22).

The rule looks like:

```
alert tcp any any -> 192.168.56.0/24 22 (msg: "NMAP TCP Scan"; sid:1000007;)
```

Save, exit, run the same test, boom, it works!

I went on to write rules looking for a few other specific NMAP scans, and they worked. The next step was thinking about how an attacker might try to avoid those rules. After all, that's the push and pull of this industry, right? 

--- Evading Snort Rules --- 

