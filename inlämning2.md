## Inlämning 2

## Man in the middle (MITM) attack

## What is it?

A man in the middle (MITM) attack is a general term for when an attacker positions himself in a conversation between a user and an application. The purpose of this is usually either to eavesdrop in order to gather sensitive information, or to impersonate one of the parties, all while making it appear as if a normal exchange of information is happening.

The goal of a MITM-attack is to steal personal information. For example login credentials, account details and credit card numbers. Targets are typically users of financial applications, SaaS businesses, e-commerce sites and other websites where logging in or providing secret information is required.

This quote nicely sums up what a MITM-attack is:
> A MITM attack is the equivalent of a mailman opening your bank statement, writing down your account details and then resealing the envelope and delivering it to your door.

## How does an attacker carry out a MITM attack?

The first step of an attack is to intercept user traffic through the attacker’s network before it reaches its intended destination.

The most common (and the most simple) way of doing this is to set up free, malicious WiFi hotspots available to the public. For example you can take your laptop to a coffee shop and set up your own network. Typically named in a way that corresponds to the location, maybe "starbucks free wifi", and without password protection. Once a victim connects to such a hotspot, the attacker gains full visibility to any online data exchange.


![image](https://user-images.githubusercontent.com/69190482/118667761-8ff47d00-b7f4-11eb-9fd8-72e5679f0da0.png)  
*How the scheme of the attack would look*  

decryption................


## How to prevent a Man in the middle attack

To prevent MITM attacks, there are a couple of practical things to do as a user. This, combined with encryption and the use of different veification methods in your applications will provide decent protection against such attacks.

Precautions needed to be taken by the user:  
- Avoid public Wifi connections that aren't protected by password . 
- Pay attention to notifications in your browser reporting a website as unsecure.
- Make sure to log out of a secure application when it's not being used.
- Do not use a public network that you do not trust while conducting more sensitive transactions.

