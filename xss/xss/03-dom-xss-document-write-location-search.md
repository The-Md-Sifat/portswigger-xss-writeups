# DOM XSS in document.write sink using source location.search

Platform: PortSwigger Web Security Academy
Category: Cross-Site Scripting (DOM-based XSS)
Difficulty: Apprentice
Goal: document.write sink ব্যবহার করে alert ফাংশন কল করানো

---

# বাংলা ভার্সন

## ১. DOM-based XSS কী?

আগের দুই ল্যাবে (Reflected ও Stored) সমস্যাটা ছিল সার্ভারে। সার্ভার ইউজারের ইনপুট এনকোড ছাড়া HTML-এ বসিয়ে দিচ্ছিল। DOM-based XSS-এ সমস্যাটা সম্পূর্ণ ভিন্ন জায়গায়, সার্ভারে নয়, ব্রাউজারের ভেতরে চলা JavaScript কোডে। পেজের নিজস্ব script URL বা অন্য কোনো ইউজার-নিয়ন্ত্রিত ডেটা পড়ে, এবং সেই ডেটা কোনো যাচাই ছাড়া পেজের ভেতরেই "লিখে" ফেলে।

এই কারণে অনেক সময় সার্ভারের response দেখলে কোনো সমস্যা ধরা পড়ে না, কারণ ইনপুটটা সার্ভার পর্যন্ত যায়ই না। পুরো প্রক্রিয়াটা ব্রাউজারের মধ্যেই ঘটে। তাই একে বলা হয় "client-side" বা "DOM-based" vulnerability।

## ২. Source ও Sink

DOM XSS বোঝার জন্য দুইটা শব্দ জানা জরুরি।

Source মানে যেখান থেকে attacker-নিয়ন্ত্রিত ডেটা আসে। যেমন location.search (URL-এর query অংশ), location.hash, document.referrer, document.cookie।

Sink মানে যেখানে সেই ডেটা বিপজ্জনকভাবে ব্যবহার হয়। যেমন document.write(), innerHTML, eval()। sink যদি ডেটাকে HTML বা কোড হিসেবে প্রসেস করে, এবং ডেটা আগে থেকে পরিষ্কার না করা হয়, তাহলে XSS ঘটে।

এই ল্যাবে source হলো location.search, sink হলো document.write।

## ৩. ল্যাবের বর্ণনা

ল্যাবের একটি search ফিচার আছে। পেজের ভেতরের একটি inline script location.search থেকে query string পড়ে এবং document.write() দিয়ে সেটি সরাসরি পেজে বসিয়ে দেয়, যাতে সার্চ বক্সে আগের সার্চ করা মান দেখানো যায়। সমাধানের শর্ত হলো এমন একটি URL বানানো, যা খুললে alert চলে।

## ৪. Recon

প্রথমে সার্চ বক্সে একটি সাধারণ শব্দ (test123) লিখে সার্চ করা হলো। URL দাঁড়াল:

```
https://LAB-ID.web-security-academy.net/?search=test123
```

Page source দেখে (Ctrl+U) খুঁজে পাওয়া গেল এই ধরনের একটি script:

```html
<script>
document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
</script>
```

এখানে query ভ্যারিয়েবলে location.search থেকে পাওয়া মান আসে। অর্থাৎ URL-এর search প্যারামিটারের মান সরাসরি একটি HTML string-এর ভেতরে জোড়া লাগানো হচ্ছে, কোনো encoding ছাড়াই।

## ৫. Exploitation

ইনপুটটি একটি img ট্যাগের src attribute-এর ভেতরে (double-quote-এর মধ্যে) বসছে। তাই সরাসরি script ট্যাগ লিখলে কাজ হবে না; আগে quote ও attribute থেকে বেরিয়ে আসতে হবে।

পেলোড:

```html
"><script>alert(1)</script>
```

এখানে `">` প্রথমে চলমান img ট্যাগটা বন্ধ করে দেয়, তারপর নতুন script ট্যাগ শুরু হয়।

এই পেলোড দিয়ে URL বানানো হলো:

```
https://LAB-ID.web-security-academy.net/?search="><script>alert(1)</script>
```

এই URL ব্রাউজারে খোলার পর script-টি চলে গেল এবং page-এ এমন HTML তৈরি হলো:

```html
<script>
document.write('<img src="/resources/images/tracker.gif?searchTerms="><script>alert(1)</script>">');
</script>
```

ব্রাউজার document.write দিয়ে এই string-কে HTML হিসেবে পেজে বসায়। তখন `">` অংশটুকু আগের img ট্যাগ বন্ধ করে দেয়, আর তার পরের script ট্যাগ আলাদা ও বৈধ একটা ট্যাগ হিসেবে চলে। alert পপআপ দেখা গেল এবং ল্যাব solved হলো।

## ৬. কেন কাজ করল?

পেজের নিজস্ব JavaScript location.search থেকে ডেটা নিয়েছে, যা সম্পূর্ণভাবে attacker-নিয়ন্ত্রিত। সেই ডেটা document.write() sink-এ পাঠানো হয়েছে, যা string-কে সরাসরি HTML হিসেবে পার্স করে। ডেটা attribute-এর ভেতরে বসেছিল বলে পেলোডে আগে quote ভেঙে বেরিয়ে আসতে হয়েছে, তারপর নতুন ট্যাগ লিখতে হয়েছে। অর্থাৎ পেলোড context অনুযায়ী বানাতে হয়। সার্ভার-সাইড কোনো filter বা encoding এখানে প্রাসঙ্গিকই না, কারণ পুরো attack ব্রাউজারের ভেতরেই ঘটেছে।

## ৭. বাস্তব ঝুঁকি

DOM XSS একই রকম বিপজ্জনক, কারণ আক্রমণকারী victim-এর browser-এ script চালাতে পারলে cookie চুরি, phishing form দেখানো, বা victim-এর হয়ে অ্যাকশন নেওয়ার মতো একই ধরনের ক্ষতি করতে পারে। পার্থক্য শুধু এটুকু যে, দুর্বলতাটা সার্ভার কোডে নয়, client-side JavaScript-এ। তাই সার্ভার-সাইড code review করলেও এই bug ধরা পড়ে না; client-side JavaScript আলাদাভাবে review করতে হয়।

## ৮. প্রতিরোধ

বিপজ্জনক sink (document.write, innerHTML, eval) এড়িয়ে নিরাপদ বিকল্প ব্যবহার করা ভালো, যেমন textContent বা createElement। document.write ব্যবহার করতেই হলে, বসানোর আগে ডেটা HTML-এর জন্য উপযুক্তভাবে encode করা দরকার। location.search বা location.hash-এর মতো source থেকে পাওয়া ডেটাকে ইউজারের ইনপুটের মতোই অবিশ্বাস করা উচিত। Content Security Policy (CSP) ব্যবহার করলে inject করা script আটকানো যায়।

## ৯. মূল শিক্ষা

XSS সবসময় সার্ভারের দোষে হয় না; ব্রাউজারে চলা JavaScript-ও কারণ হতে পারে। source আর sink চেনা DOM XSS খোঁজার মূল পদ্ধতি। পেলোড কোন HTML context-এ বসছে (body, attribute, script) সেটা আগে বোঝা জরুরি, তারপর সেই অনুযায়ী পেলোড বানাতে হয়। Page source দেখলেই DOM XSS সবসময় ধরা পড়ে না; মাঝে মাঝে browser DevTools দিয়ে JavaScript ফাইল আলাদা করে দেখা লাগে।

---

# English Version

## 1. What is DOM-based XSS?

In the previous two labs (Reflected and Stored), the flaw was on the server: it placed user input into HTML without encoding. DOM-based XSS is different; the problem is not on the server but inside the JavaScript running in the browser. The page's own script reads some user-controlled data (such as the URL) and writes it back into the page without sanitization.

Because of this, the server's response often looks completely safe, since the input never actually reaches the server. The whole process happens inside the browser, which is why this is called a "client-side" or "DOM-based" vulnerability.

## 2. Source and Sink

Two terms are essential to understand DOM XSS.

Source means where attacker-controlled data comes from. Examples: location.search (the query part of the URL), location.hash, document.referrer, document.cookie.

Sink means where that data is used in a dangerous way. Examples: document.write(), innerHTML, eval(). If a sink processes the data as HTML or code, and the data was not sanitized beforehand, XSS occurs.

In this lab, the source is location.search and the sink is document.write.

## 3. Lab Description

The lab has a search feature. An inline script on the page reads the query string from location.search and writes it directly into the page using document.write(), so the search box can display the previously searched value. The goal is to craft a URL that, when opened, triggers alert.

## 4. Recon

A simple string (test123) was searched first. The URL became:

```
https://LAB-ID.web-security-academy.net/?search=test123
```

Viewing the page source (Ctrl+U) revealed a script similar to this:

```html
<script>
document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">');
</script>
```

Here, the query variable holds the value from location.search. The value of the URL's search parameter is being concatenated directly into an HTML string with no encoding at all.

## 5. Exploitation

The input lands inside the src attribute of an img tag (inside double quotes). So writing a script tag directly will not work; the quote and the attribute must be broken out of first.

Payload:

```html
"><script>alert(1)</script>
```

Here, `">` first closes the ongoing img tag, and then a new script tag begins.

The URL built with this payload:

```
https://LAB-ID.web-security-academy.net/?search="><script>alert(1)</script>
```

Opening this URL ran the script, which produced this HTML on the page:

```html
<script>
document.write('<img src="/resources/images/tracker.gif?searchTerms="><script>alert(1)</script>">');
</script>
```

The browser writes this string into the page as HTML via document.write. The `">` part closes the earlier img tag, and the following script tag becomes a separate, valid tag that runs. The alert popup appeared and the lab was solved.

## 6. Why Did It Work?

The page's own JavaScript read data from location.search, which is entirely attacker-controlled. That data was passed into the document.write() sink, which parses the string directly as HTML. Because the data landed inside an attribute, the payload first had to break out of the quote before writing a new tag. The payload had to match its context. Server-side filtering or encoding is irrelevant here, because the entire attack happens inside the browser.

## 7. Real-World Impact

DOM XSS is just as dangerous, since running a script in the victim's browser enables the same kinds of harm: cookie theft, phishing forms, or performing actions on the victim's behalf. The difference is only that the flaw lives in client-side JavaScript, not server code. Reviewing server-side code alone will not catch this bug; client-side JavaScript needs to be reviewed separately.

## 8. Mitigation

Avoid dangerous sinks (document.write, innerHTML, eval) and use safer alternatives such as textContent or createElement. If document.write must be used, properly HTML-encode the data before inserting it. Treat data from sources like location.search and location.hash with the same distrust as any user input. A Content Security Policy (CSP) can help block injected scripts from executing.

## 9. Key Takeaways

XSS is not always the server's fault; JavaScript running in the browser can be the cause too. Recognizing sources and sinks is the core method for finding DOM XSS. It is important to first identify which HTML context the payload lands in (body, attribute, script), and craft the payload accordingly. DOM XSS is not always visible in the page source; sometimes the browser's DevTools are needed to inspect the JavaScript file separately.

---

## Screenshot



![solved](../images/lab3-solved.png)
