- - - 
created : 24-09-2025 

Tags : #easy 

Category : OSINT

Released on 20/09/2025
- - - 
# CHALLENGE DESCRIPTION

A sophisticated **cryptocurrency fraud** operation has victimized multiple individuals over a 4-month period. The San Francisco Police Department Cyber Crimes Unit has seized blockchain evidence and requires comprehensive forensic analysis using professional-grade tools to identify perpetrators, trace fund flows, and attribute real-world identities.
# Primary Funding Source Exchange

**Analyze transaction clusters to identify the primary funding source exchange.**
## CHAINALYSIS: Entity attribution through compliance databases

We need to copy the primary suspect address in the `Case Overview` page:

```
Primary suspect address isolated: {{REDACTED}}
```

Now we go to the `Tools` page and we click on `Chainalysis KYT` then enter the address and `INVESTIGATE` button:


## CRYSTAL: Network analysis for funding source verification 

We analyze the same address with `Crystal Analytics`:


# 🔒 Locked Content

This Challenge is currently **Active** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need the full write-up you can **DM ME !!!** 


## ARKHAM: Intelligence cross-referencing for exchange confirmation

Now we use `Arkham Intelligence`, same address:



We can answer the first question:


# Total Victim Payment Amount

**Calculate the total victim payment amount from INVESTIGATION CLUSTER CHARLIE in the Evidence Terminal. Add up all 7 confirmed victim payments (T1-CHARLIE through T7-CHARLIE).**
## BLOCKSCOUT: Detailed blockchain transaction data and payment verification

We use the `BlockScout Enterprise` tool with the given address:



We can check the previous results on `Crystal` too for the answer:


# Obfuscation Method Detection

**Identify the mixing service protocol used. Use OXT Advanced with the transaction hash: `{{REDACTED}}`**
## OXT Advanced

We copy the given `hash` and analyze it with `OXT` tool:




We can combine these with the previous `Chainalysis` results for the answer:

```

```

# Digital Identity Attribution

**Determine the email address associated with the primary suspect through KYC analysis.**
## KYC Analysis

**Extracting data from previous scans**

From `Chainalysis`

```
{{REDACTED}}
```

From `Crystal`:

```
{{REDACTED}}
```

From `Arkham`:

```
{{REDACTED}}
```

We can group these info as follows:

```
{{REDACTED}}
```
## Digging

We find another exchange address here that works:



We can use `BlockScout`:





`{{REDACTED}}` is probably the `CASH-OUT EXCHANGE`. 

The others addresses don't give results on any tool:


Tried the `Suspicious Pattern Addresses` too on each tool, no-one works:


# Real-World Identity Attribution

**Determine the primary suspect's real name through comprehensive analysis.**

We already know the full name after our `KYC analysis`: 


# Geographic Attribution

Determine precise operational coordinates through comprehensive temporal and geographic analysis.
## Evidence Terminal + OXT

We can find the `{{REDACTED}}` hash:


Let's run `OXT`:



We have already seen the same coordinates on `Arkham` in the first analysis, we can confirm these results. 

The other found hash is `{{REDACTED}}`, doesn't seems to work.

We can answer the last question and get the flag.

![Pasted image 20250924153345.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250924153345.png)

Thanks for reading, hope you enjoyed :).

