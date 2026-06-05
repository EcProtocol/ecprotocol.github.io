---
layout: post
title: "Out of sight, out of reach"
date: 2026-06-05
author: Lars Szuwalski
---

We all know it: Platforms feed on our data-streams. They train AI models from it and they compile comprehensive marketing profiles on us. To varying degrees of course. But what was actually in the fine print on all those *terms of service* that I just mindlessly "Accepted" to use the product?
As they say: if you are not paying for the product - then *you are the product*. And sometimes even if you pay.

Let's take a few examples:
- Gmail, "we do not read your mail to serve you ads" but if you pay for Workspace, "we will not train Gemini on your data" - so what if I don't pay? Is there a sentence on that buried deep in the wall of text? Maybe. I live in Denmark where schools got in trouble for handing out Chromebooks to kids because of unclear boundaries and guarantees.

- Meta is more direct. The products, Facebook, Instagram are ad platforms - and your data is "public" anyway... at least some of it. So anyone (if they would allow scraping) could do it - but *they* saw it first.

- As a third example take card-payments. The large duopoly Visa and Mastercard are pretty straight about it: Yes, analytics, modeling, AI and "product development" happen on your data (and let me add you also pay for their services). Maybe Mastercard more than Visa. But smaller schemes like American Express are also in the game. It's business.

The common theme of course is that inspecting your data brings benefits to you as a user. Who doesn't want to fight card-fraud, email-spam or hateful posts on social media? That obviously only works if we can see the data - and we will also need to know "who you are" such that we can sanction you in case of a breach of terms.

## So what are the alternatives?
A public post like this one is sort of "fair game". I even put my name on it. If I didn't want people and machines to notice my opinions I could have stayed under the tinfoil hat. But my post in that group of parents on Facebook? My email? Or my payments?

Eventually Fully Homomorphic Encryption (FHE) could save the day. Still want to look for spam or fraud in my data without actually reading it? Yeah, some day. Maybe even possible to run inference from an encrypted query - so welcome Private-ChatGPT!

But let's be honest. It will not be free on the platforms: Increased compute and now limited possibilities to "extract" other value from the data. And you better read up on your math degree if you want to be really sure what is possible given a specific encoding. But it's probably the holy grail for some use cases. When it scales.

End-to-end encryption of course could also shield your data from intermediaries. So that works fine if the platform is built like that. But the metadata and key agreements in groups? And again: Only while the platform stays like that. Everyone and their mom can put "end-to-end encrypted" on the box. But what's really *in* the box is a bit of another story.

So let me present another comparatively stupid and simple alternative, which I think matches how "not so tech oriented" people assume things work today: Hand the data directly to the intended receiver. Do not mail your letter without an envelope - or better yet: Deliver it yourself. Out of sight, out of reach.

## It's the UX, stupid
Well, how is that supposed to work then, you may ask. In this vast digital universe, where should I drop my letter to you? And if I pay you with a digital "note" (it even contains a signature from a real bank and all), what's stopping me from giving the exact same bytes to another merchant? In fact: Let's post it on the Internet, so more people can share it.

So are we talking Peer-to-Peer (P2P)? That doesn't seem to solve any of those issues above, right? Adding on top of "what if you are not home/on"? So even if you rented an "endpoint" where I could deliver my mail - I still need to find you. So you could publish the IP/Port and the type of service (here is my "phone number"). Tricky if you need to change it later. So DNS then, that's old as the Internet itself and *just works (tm)*. But the UX and the cost? And each of my kids and the wife need an entry too? Okay, circle back to self hosting a mail server and a domain. Spammers ruined that one, didn't they? Practically only established mail service providers are accepted in that game.

So let's say that we actually had a shared and open service where I could bind a SHA of my name to an IP/Port of that service. Then you would be able to resolve "lars szuwalski" or whatever handle to an endpoint. Why not just the name? Of course I can make the service reject letters from unknown senders - but I could also not post the address in plain-text on the Internet. Let's say I could update it when I got tired of one host and wanted to move to another. If it was really cheap to "write" in this registry I could easily change and add as I liked.

Then what about the payment? Digital money would be easy - but you need to know who owns a note at any point in time. So imagine the service above could map the SHA of that note to, say, a public key. And to hand it over to someone else would be signing the update of that entry with the matching key - and putting your public key on the note. And we just agreed it would be easy and cheap to update. So that could work for tickets and other such items as well, right?

The point is that current architectures don't work like that. And the UX on established systems have been polished to the point where you don't even notice the pill when you swallow. But those architectures are not handed down by God. Alternatives could be built. We do not *need* to walk around digitally naked.

Look at the footer: Well, it's not working yet - so I'm not here to sell you anything. Just wanted to share my thoughts. Thanks for reading this far. Have a nice weekend!
