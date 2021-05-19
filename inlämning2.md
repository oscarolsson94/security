## Inlämning 2

## Man in the middle (MITM) attack

## What is it?

A man in the middle (MITM) attack is a general term for when an attacker positions himself in a conversation between a user and an application. The purpose of this is usually either to eavesdrop in order to gather sensitive information, or to impersonate one of the parties, all while making it appear as if a normal exchange of information is happening.

The goal of a MITM-attack is to steal personal information. For example login credentials, account details and credit card numbers. Targets are typically users of financial applications, SaaS businesses, e-commerce sites and other websites where logging in or providing secret information is required.

This quote nicely sums up what a MITM-attack is:
> A MITM attack is the equivalent of a mailman opening your bank statement, writing down your account details and then resealing the envelope and delivering it to your door.

## How does an attacker carry out a MITM attack?

The first step of an attack is to intercept user traffic through the attacker’s network before it reaches its intended destination.

The most common (and the most simple) way of doing this is to set up free, malicious WiFi hotspots available to the public. For example you can take your laptop to a coffee shop and set up your own network. Typically named in a way that corresponds to the location, something along the lines with "starbucks free wifi", and without password protection. Once a victim connects to such a hotspot, the attacker gains full visibility to any online data exchange.


![image](https://user-images.githubusercontent.com/69190482/118667761-8ff47d00-b7f4-11eb-9fd8-72e5679f0da0.png)  
*How the scheme of the attack would look*  

Even though the perpetrator now has access to all the data, most of todays sensitive data will be encrypted in some way, which means it has to be decrypted in order to be of use. Depending on your security level and precautions taken however, that might or might not be easier than you think. There are multiple ways of decrypting two-way SSL (Secure Sockets Layer) traffic, here are a few options:  

- **HTTPS spoofing** sends a fake certificate to the victim’s browser once the initial connection request to a secure site is made. This certificate holds a digital thumbprint associated with the compromised application, which the browser verifies according to an existing list of trusted sites. The attacker is then able to access any data entered by the victim before it’s passed to the application.
- **SSL BEAST** (browser exploit against SSL/TLS) targets a TLS version 1.0 vulnerability in SSL. Here, the victim’s computer is infected with malicious JavaScript that intercepts encrypted cookies sent by a web application. Then the app’s cipher block chaining (CBC) is compromised so as to decrypt its cookies and authentication tokens.
- **SSL hijacking** occurs when an attacker passes forged authentication keys to both the user and application during a TCP handshake. This sets up what appears to be a secure connection when, in fact, the man in the middle controls the entire session.
- **SSL stripping** downgrades a HTTPS connection to HTTP by intercepting the TLS authentication sent from the application to the user. The attacker sends an unencrypted version of the application’s site to the user while maintaining the secured session with the application. Meanwhile, the user’s entire session is visible to the attacker.


## How to prevent a Man in the middle attack

To prevent MITM attacks, there are a bunch of practical things you can do as a user. This, combined with encryption and the use of different verification methods in your applications will provide decent protection against such attacks.

Precautions needed to be taken by the user:  
- Avoid public Wifi connections that aren't protected by a password. 
- Pay attention to notifications in your browser reporting a website as unsecure.
- Make sure to log out of a secure application when it's not being used.
- Do not use a public network that you do not trust while conducting more sensitive transactions.


For website owners or application developers, secure communication protocols, including TLS and HTTPS, help protecting against spoofing attacks by robustly encrypting and authenticating transmitted data. Doing so prevents the attacker from intercepting site traffic and blocks the decryption of sensitive data, such as authentication tokens.

It is considered best practice for applications to use SSL/TLS to secure every page of their site and not just the pages that require users to log in. Doing so aids in decreasing the chance of an attacker stealing session cookies from a user browsing on an unsecured section of a website while logged in.
  
## What can we learn and how does this correlate to the course material?  
  
Be careful when connecting to random Wifi hotspots or networks that you dont trust. You should always be aware of the risks and possible consequences of sending private information over the internet while connected to an unknown network. Avoid logging in to personal accounts or submitting sensitive information while on an unknown network, especially things like online banking, entering credit card information and things alike.

This attack correlates to the course content in the way that it is partly software related. Your applications encryption and security plays a part in how difficult your information is for the attacker to harvest. However, the major part of this vulnerability comes down to human error or just simply lack of knowledge from users. I believe this issue needs to be brought up in order for more people to realize the severe damage this attack could cause, without you even knowing what or how it could have happened. 
  
## Final Note  

Use free, public wifi on your own risk. You dont always have the luxury of connecting to a network which you know is completely safe, for example when travelling. In this case, be aware of what possible threats are out there, and protect yourself in the ways that you can. As a developer, use SSL to secure your entire application or every page of your website, not just the pages handling login.
