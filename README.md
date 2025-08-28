# TryHackMe - Pyrat Writeup - (Chaos Path)


## Recon

### Step 1: The Overkill Scan

Like a responsible nerd, I kicked things off with:

```bash
sudo nmap -A -p- -vv 10.10.xxx.xxx -oA pyrat-scan
```

…and instantly regretted it. Full port sweep + OS detect + script scans = absolute sloth mode. Watching it crawl was like babysitting a toaster.

### Step 2: The “Stuff This” Scan

Couldn’t be bothered waiting, so I bailed and ran a lighter scan just to see *something* quick:

```bash
nmap 10.10.xxx.xxx
```

That spits out:

* **22/tcp** → SSH
* **8000/tcp** → some kind of web service

Didn’t know it was Python at this point — no `-sV` = no service banner. Just knew there was something alive on 8000.

## Step 3: Beating the Dead Page

So I hit up `http://10.10.xxx.xxx:8000` in browser. Nothing. Dead.

Try `curl`:

```bash
curl http://10.10.xxx.xxx:8000
```

Curl just sits there blank, doing the whole nothing-burger routine. I spend **30 mins** smashing refresh like an idiot waiting for the miracle page. Nada.

## Step 4: “More Basic Connection” (Netcat Brainwave)

The room hint says “try a more basic connection.” I’m like *more basic than curl, what, crayons??*

So I throw:

```bash
nc 10.10.xxx.xxx 8000
```

At first it just sits there — no banner, no clue. I don’t even know if NC is “listening” or what. Is it a server? Is it a fake HTTP? Is it just vibes?

Finally type `Hello` and boom — Python screams back:

```
name 'Hello' is not defined
```

Eval server confirmed.


## Step 5: The Git Jackpot

Poking around eventually reveals a `.git` folder in `/opt/dev`.

```bash
cd /opt/dev/.git
cat logs
cat config
```

The config shows a GitHub username. I yeet it into Google, find a Pyrat repo. It’s the actual source code. Jackpot.

I see the backdoor logic: there’s an `admin` command in the code. Sweet.


## Step 6: Broken Admin Command

Jump back into NC, type `admin`. Nothing. Doesn’t work. Legit broken.

Spend ages chasing my tail, thinking maybe I’ve got the wrong flow. Off on a goose chase.


## Step 7: Wait… I Haven’t Even Got User Flag

Lightbulb moment: *I’ve been chasing root like a clown and skipped the user flag completely.*

Search around, still nothing obvious. Crawl back into the original `.git` folder, this time actually read `config`.

See creds for user **think**.


## Step 8: SSH With Think

```bash
ssh think@10.10.xxx.xxx
```

Boom, works.

Check home dir, realise yep — I skipped the flag earlier. ADHD gremlin moment.

```bash
ls -lta
cat user.txt
```

User flag in the bag.

Go eat dinner 


## Step 9: Dinner Reset

Come back, machine’s dead. Restart it.

NC back into 8000:

```bash
nc 10.10.xxx.xxx 8000
```

Type `admin` again — and THIS time it actually prompts:

```
Password:
```

Internal screaming. *Where was this prompt hours ago?!*

---

## Step 10: Brute Force Victory

Thank god I’ve got a coding robot in my pocket - smashed out a Python brute script with timeouts, markers, fresh reconnects each attempt. Let ChatGPT chew the pain, and finally… bang, password drops.

Password pops:

```
abc123
```

Clean. Script actually did its job. Felt like I’d just tamed a wild donkey.


## Step 11: Root

SSH in as root with creds.

```bash
ssh root@10.10.xxx.xxx
cat /root/root.txt
```

Root flag down. Machine pwned. Sanity hanging by a thread, but full clear achieved.


## Reflections

* Box flow is simple: eval backdoor → `.git` creds → admin command → brute pw → root.

* My flow:

  * Overkill nmap → rage quit → lighter nmap
  * 30 mins refreshing a dead curl page
  * Eval shenanigans with NC
  * `.git` raccoon moment
  * Skipped user flag
  * Dinner noodles
  * NC finally behaves
  * Fixed the brute script, popped `abc123`
  * Root via SSH

* **Verdict:** me = cooked, machine = probably scuffed in places, but I actually took it all the way through this time.
