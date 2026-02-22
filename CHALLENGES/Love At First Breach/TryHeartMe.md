# TryHeartMe

# **TryHeartMe**

## **Objective**

Exploit business logic vulnerabilities to escalate privileges and retrieve a hidden flag.

## **Steps Performed**

1. **Initial Analysis**
    - Understood from the room description that the challenge focuses on business logic flaws.
    - Explored all application functionalities to map potential attack surfaces.
2. **Account Creation and Token Inspection**
    - Created a user account.
    - Intercepted a request to `/account` and observed a JWT token in the authorization header or cookie.
    - Decoded the token using [jwt.io](https://jwt.io/) and found it contained a key-value pair for `role` and `credits`.
3. **Token Manipulation**
    - Modified the token by changing the `role` value from `user` to `admin`.
    - Encoded the altered token (keeping the original signature or using a known weak secret if applicable).
4. **Exploiting the Purchase Functionality**
    - Used the manipulated JWT in a `POST` request to `/buy/rose-bouquet`.
    - Changed the product parameter from `rose-bouquet` to `valenflag` as hinted in the introductory letter.
    - Sent the request and received a redirect response.
5. **Flag Discovery**
    - Followed the redirect in Burp Suite.
    - In the response, clicked the **Render** option to view the page content.
    - Switched to the **Pretty** tab and searched for the string `THM` to locate the flag.
    - Copied the flag.