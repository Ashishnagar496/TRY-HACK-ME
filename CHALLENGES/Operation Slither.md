# Operation Slither

---

# Operation Slither – Complete Walkthrough Guide

**Room Link:** [TryHackMe | Operation Slither](https://tryhackme.com/room/operationslitherIU)**Difficulty:** Easy

### **Objective**

Follow the digital trail of the "Sneaky Viper" group to identify its members and uncover their secrets. Use open-source intelligence (OSINT) techniques to track down three operators based on forum posts, social media interactions, and code repositories.

---

## Task 1: The Leader

**Introduction**
We've gained access to a hacker forum and found our company's data for sale. All we have to start with is this post. Your mission is to find any information related to the group's leader.

**The Post**

```
Full user database TryTelecomMe on sale!!!

As part of Operation Slither, we've been hiding for weeks in their network and have now started to exfiltrate information.
This is just the beginning. We'll be releasing more data soon. Stay tuned!

@v3n0mbyt3_
```

**Reconnaissance Guide**

- Start with the provided username and search across common social platforms.
- Correlate any discovered profiles to confirm they belong to the same person.
- Carefully review their interactions, posts, and replies for more clues.

### Steps & Answers

-
--Step 1: Finding the Leader's Other Platform--

1. Search for the username `v3n0mbyt3_` on Google.
2. The first result should lead you to a popular social media platform. Look for the platform's name in the search result or on the profile page itself.

> **Question 1 Answer:** `instagram`
-
> 

-
--Step 2: Finding the First Flag--

1. On the `v3n0mbyt3_` Instagram profile you found in Step 1, navigate to the **replies** or comments section under their posts.
2. Look for a reply that contains a string of text that appears to be encoded. This is often a **base64 string** (it will look like a random mix of letters, numbers, and equals signs).
3. Copy this string and decode it using a base64 decoder (you can use an online tool or a command-line tool like `base64 -d`).

> **Question 2 Answer:** The decoded string is the flag.
-
> 

---

## Task 2: The Sidekick

**Introduction**
A second message has appeared, but our access to the forum has been revoked. Follow the leads from the first task to identify the second operator.

**The Post**

```
60GB of data owned by TryTelecomMe is now up for bidding!

Number of users: 64500000 Accepting all types of crypto
For takers, send your bid on Threads via this handle:

HIDDEN CONTENT
```

**Reconnaissance Guide**

- Use the usernames and connections you found in Task 1 to expand your search.
- Look for linked accounts on other platforms and examine any shared content.
- Follow links to media or external resources mentioned in profiles or posts.

### Steps & Answers

-
--Step 1: Finding the Second Operator's Username--

1. Return to the platform where you found the leader (`v3n0mbyt3_`).
2. Look again at the replies or comments section. You should find another user who is actively replying to or interacting with the leader. This user's handle is the answer.

> **Question 1 Answer:** The username of the replying user.
-
> 

-
--Step 2: Finding the Second Flag--

1. Go to the Instagram profile of the second operator you just identified.
2. Look through their posts for a **reel** (a short video). Watch the reel and check its **comments section**.
3. In the comments, you should find a link to an external platform, likely a music streaming service like **SoundCloud**.
4. Visit the linked profile on SoundCloud. Look at their playlists or "prototypes." Click on the **second prototype** listed.
5. Within that prototype's description or comments, you will find another encoded string (likely base64). Decode it.

> **Question 2 Answer:** The decoded string is the flag.
-
> 

---

## Task 3: The Last Operator

**Introduction**
A new post has been made. Your goal is to find the third operator using your past discoveries and uncover details about their attack infrastructure.

**The Post**

```
FOR SALE

Advanced automation scripts for phishing and initial access!

Inclusions:
- Terraform scripts for a resilient phishing infrastructure
- Updated Google Phishlet (evilginx v3.0)
- GoPhish automation scripts
- Google MFA bypass script
- Google account enumerator
- Automated Google brute-forcing script
- Cobalt Strike aggressor scripts
- SentinelOne, CrowdStrike, Cortex XDR bypass payloads

PRICE: $1500
Accepting all types of crypto
Contact me on REDACTED@protonmail.com
```

**Reconnaissance Guide**

- Find secondary accounts by checking visible interactions like likes, follows, or collaborations from previously identified operators.
- Extend your search to developer or technical platforms (like GitHub) associated with the identity.
- Analyze their activity history, such as code repositories and commits, for hidden information.

### Steps & Answers

-
--Step 1: Finding the Third Operator's Handle--

1. Go back to the **SoundCloud** profile of the second operator from Task 2.
2. Check the list of users they are **following**. Look through this list for a username that fits the pattern of the group (likely something cryptic or "hacker-styled").
3. This username is the handle of the third operator.

> **Question 1 Answer:** The username found in the second operator's SoundCloud following list.
-
> 

-
--Step 2: Finding the Third Operator's Other Platform--

1. Search for the third operator's handle on Google.
2. Among the search results, you will find their profile on a popular code-sharing and development platform.

> **Question 2 Answer:** The name of this platform, in lowercase.
-
> 

-
--Step 3: Finding the Third and Final Flag--

1. Go to the code repository platform profile you just identified. Look through their repositories.
2. Find a repository related to "automation" or "users." Open it.
3. Check the repository's **commit history**. Look for a commit that might contain a password or a secret. This could be in the commit message or in the code diff of a specific commit.
4. You will find a value that looks like another encoded string (likely base64). Decode it to reveal the final flag.

> **Question 3 Answer:** The decoded string is the flag.
-
> 

---

### **Congratulations!**

You have successfully tracked down all three operators of the Sneaky Viper group by following their digital footprints across social media, music platforms, and code repositories. You have mastered basic OSINT techniques!

I hope this formatted guide is exactly what you were looking for. It provides a clear, step-by-step path through the room while maintaining all the specific details and answers from your original notes.