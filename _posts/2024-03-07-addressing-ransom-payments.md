---
layout: post
title: Should We Outlaw Ransom Payments?
---

Over the past couple months, I've observed or taken part in multiple discussions about whether to ban ransom payments for data extortion. Discussion like these, including [one](https://x.com/ImposeCost/status/1764568502680133638?s=20) started by [@ImposeCost](https://twitter.com/ImposeCost) earlier this week, are always important. In particular, many people get locked into a frame of reference that ignores the realities faced by other organizations. Some companies are too small to recognize the value or necessity of Incident Response planning until after they've been targeted. Some assume their MSSP is positioned to completely prevent these types of attacks, despite a lack of coverage on legacy hardware or other potential issues. 

# What Are They Paying For?

In reality, the takedown against LockBit taught us a few key points that all professionals involved in IR engagements need to consider and remind victims of. First, we now know that these threat actors aren't being truthful about the protection of victim data. LockBit maintained copies of numerous victims who paid the ransom, despite promising that their exfiltrated data would be deleted. We also know that plenty of victim details are maintained by these threat actors after the fact, which could lead affiliates to target previously-responsive victims through their affiliate relationship with other Ransomware-as-a-Service groups. 

So, really, all most victims can rely on is the decryption tool, assuming that it works as expected against servers and workstations in their environment. 

# Banning Ransom Payments?

There is potential value in banning ransom payments at a state or national level. Ransomware groups are only interested in conducting offensive operations to earn money. The biggest revenue source for these groups are the payments made by victims in exchange for the decryption tool and the deletion of sensitive corporate, employee, or customer data. If a government ban is imposed on these payments, ransomware groups are likely to lose the vast majority of this revenue source. 

## But what if a victim has no choice but to pay for the decryption tool?

To me, this is probably the most important question. Whether a ban is in place or not, there will be organizations with no choice but to pay for decryption or lose everything. Because there's going to be a demand for decryption, there will be individuals or groups available to fill that demand. In my mind, the simplest example will be third-party groups claiming to be capable of building single-use decryption software, who actually just negotiate and pay the ransomware group on the victim's behalf, taking the decryption tool back to their client and claiming it is a custom-built tool. 

We've already seen similar actions taken by recovery and IR firms before, with groups reaching out via the victim chat link over Tor and negotiating, then providing a price to their client that's significantly higher than the ransomware group's demand. An important factor here is that the third-party group would likely have to remain vague about their methodology for creating the decryption tool, and would not be from a country that's prohibited from paying a ransomware group's demands. 

<p align="center">
  <img src="../public/2024-03-07/ransom_payment.jfif" />
</p>


# Is Banning Ransoms Enough?

Obviously, this potential for third-party groups to pop up offering decryption capabilities assumes that ransomware operations are still ongoing against victims in countries with bans in place. Why would ransom operations be continuing against victims who are prohibited from paying?

This depends on whether the outcomes of ransomware encryption are still favorable to the ransomware operators, and whether the victims are being intentionally targeted. 

## Favorable Outcomes

By favorable outcomes, I mean are there other benefits these ransomware groups get from victimizing a group besides payment of the ransom. 

- These groups steal data in most cases, so maybe they shift their operations to attempt to collect more valuable information from their victims, leading to data that other entities are more willing to pay for. 
- They could continue to hit high visibility groups for the notoriety and to attempt to pressure those governments into making exceptions to their policy, resulting in payments. 
- Foreign government entities and their proxies could pay various ransomware actors to target key organizations to exfiltrate important data and impact operations without directly tying to any nation-state.

## Unintentional Targeting

With mass exploitation campaigns, like Clop's attacks against MOVEit, and recent attacks from multiple groups against Ivanti Connect Secure and ConnectWise ScreenConnect, we've seen that many attacks are conducted with little or no care for the nature of their victims. Groups observe a method of mass exploitation and execute it as quickly as possible, turning back to see which victims they've hit and attempt to squeeze value out of the most important targets after the fact. These attacks, while possibly not intentional against victims in countries with payments bans in place, would still occur, leading to disastrous effects against those victims with no means to recover. 

# There are Good Arguments for the Bans

I'm not saying all of this because I disagree with banning ransomware payment. Personally, I think CISA and other government entities could do a lot more to provide direct guidance on how to protect against both the execution of ransomware against sensitive networks and the effects of ransomware executed against these victims. The [#StopRansomware](https://www.cisa.gov/stopransomware) program includes great advice and up-to-date information about the latest ransomware threats. Their [Cyber Hygiene Services](https://www.cisa.gov/cyber-hygiene-services) provide free scanning of government and critical infrastructure organizations to help identify risks relevant to ransomware engagements.  Imposing regulatory requirements for data protection and prohibiting payments to ransomware actors, particularly if the organization did not hold up to the relevant portions of those regulations can create reasonable expectations among the industries to help protect against the need for ransom payments at all. 

Organizations should make every effort to avoid payment of these ransoms. They need to consider the limited value ransomware groups get from holding the exfiltrate data, and include the fact that they hold the data regardless of whether their victims pay. The only value payment provides is that the ransomware group is more likely to keep the data off of their leak site, but with the proliferation of ransomware across all industries, the stigma of being hit by ransomware is far less meaningful than it used to be.

<p align="center">
  <img src="../public/2024-03-07/do_not_pay.jfif" />
</p>

# In Conclusion

I need to figure out how to write my blog in a more interesting way... I feel like that was pretty boring, but I can't distill thoughts down to 280 characters like so many of the thought leaders in this space. Hit me up on X with additional thoughts or feedback!