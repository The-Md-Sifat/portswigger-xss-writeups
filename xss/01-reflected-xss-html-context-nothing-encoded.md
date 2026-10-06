# Reflected XSS into HTML context with nothing encoded

| Item | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Category | Cross-Site Scripting (Reflected XSS) |
| Difficulty | Apprentice |
| Goal | Make the browser call the `alert` function |
| Tools | Browser (solvable without Burp Suite) |

---

## 1. What is XSS?

XSS stands for Cross-Site Scripting. It is a vulnerability that allows an attacker to inject their own JavaScript into a website's page, which then runs in the victim's browser as if it were the website's own code.

Naming history: it was originally abbreviated as CSS. Since Cascading Style Sheets also uses the abbreviation CSS, the vulnerability was renamed to XSS to avoid confusion. The term "cross-site" comes from the fact that the attack originates from another source (a link sent by the attacker) and executes inside the vulnerable site.

There are three main types of XSS:

| Type | Characteristics |
|---|---|
| Reflected XSS | The payload is part of the request and is immediately reflected in the response. It is not stored anywhere. |
| Stored XSS | The payload is saved on the server (for example, in a database) and affects every user who views the page. |
| DOM-based XSS | The flaw is not on the server but in the client-side JavaScript. |

This lab is the simplest example of the first type, Reflected XSS.

---

## 2. Lab Description

The lab website has a search feature. Whatever is typed into the search box is placed into the response page by the server without any filtering or encoding. To solve the lab, a payload must be sent that causes the `alert` function to execute when the page loads.

---

## 3. Recon

### Step 1: Testing with a harmless string

A unique, easily recognizable string (`test123`) was entered into the search box. A unique string is used so that it can be located quickly in the page source with `Ctrl+F`.

### Step 2: Observing the URL change

After the search, the browser URL changed to:

```
https://LAB-ID.web-security-academy.net/?search=test123
```

The parts of this URL are:

| Part | Meaning |
|---|---|
| `?` | Marks the start of the query string |
| `search` | The name of the parameter |
| `=` | Separator between the name and the value |
| `test123` | The parameter's value, i.e. the user's input |

This shows that the search input is sent to the server through a GET request as a parameter named `search`. Because of this, changing the value in the URL changes the input. The payload can also be placed directly in the URL instead of the search box, which is exactly what makes Reflected XSS suitable for delivery through a link.

### Step 3: Inspecting the page source

The page source was opened (right-click, View Page Source, or `Ctrl+U`) and `test123` was searched for. The string appeared inside an HTML `<p>` tag:

```html
<p>test123</p>
```

(This is simplified. The real page may contain more text around the tag.)

### Step 4: Checking whether encoding is applied

In HTML, `<` and `>` are special characters because they open and close tags. A secure website converts (encodes) them into `&lt;` and `&gt;`, so the browser treats them as plain text instead of tags.

In this lab, no encoding is applied. The characters entered by the user are placed into the HTML exactly as typed. This means that if HTML tags are entered as input, the browser will parse them as real tags.

(Optional check: entering `<b>test</b>` in the search box and seeing the text rendered in bold confirms that the input is processed as HTML.)

---

## 4. Exploitation

### Payload

```html
<script>alert(1)</script>
```

This is a simple proof-of-concept payload. The `<script>` tag tells the browser to run JavaScript, and `alert(1)` displays a popup. `alert` is used because it is harmless yet clearly proves that attacker-controlled code is executing.

### Sending the payload

The payload was entered into the search box and submitted. The resulting URL was:

```
https://LAB-ID.web-security-academy.net/?search=<script>alert(1)</script>
```

The browser automatically percent-encodes special characters in the URL before sending:

```
?search=%3Cscript%3Ealert(1)%3C/script%3E
```

Here `%3C` represents `<` and `%3E` represents `>`. The server decodes this and obtains the original value.

### Result

The server inserted the payload inside the `<p>` tag and returned the response. The page HTML became:

```html
<p><script>alert(1)</script></p>
```

While parsing the page, the browser encountered the `<script>` tag and executed the code inside it. The `alert` popup appeared, and the lab displayed the message "Congratulations, you solved the lab!"

---

## 5. Why Did It Work?

Step by step:

1. The server placed the user's input into the response HTML without any sanitization.
2. The `<` and `>` characters in the input were not encoded.
3. The browser's HTML parser cannot tell which part was written by the developer and which was injected by the user. It reads the whole response as trusted HTML.
4. The input landed in the HTML body (an HTML context), where a `<script>` tag is directly valid. Therefore no extra technique was needed, such as breaking out of quotes or escaping an attribute.
5. The browser runs the script as part of the website's origin, so it has access to that site's cookies, session, and DOM.

---

## 6. Real-World Impact

`alert(1)` is only a proof of concept. In a real attack, the same path can be used to:

- Steal session cookies and take over the victim's account (session hijacking)
- Display a fake login form to capture passwords (phishing)
- Install a keylogger to capture what the user types
- Perform actions on behalf of the victim (for example, changing the password or email)

In Reflected XSS, the attacker must trick the victim into clicking the crafted link, for example via email, messages, or social media. This is its main limitation compared to Stored XSS.

---

## 7. Mitigation

| Measure | Explanation |
|---|---|
| Output encoding | Encode output according to its context: `<` to `&lt;`, `>` to `&gt;`, `"` to `&quot;`, and so on. This is the most important defense. |
| Content Security Policy (CSP) | Restrict which sources scripts may run from. This blocks injected inline scripts. |
| HttpOnly cookies | Prevent JavaScript from reading session cookies, making cookie theft harder. |
| Input validation | Reject input that does not match the expected format. This is not sufficient alone and complements encoding. |
| Secure frameworks | Modern frameworks (such as React) escape output by default. |

---

## 8. Key Takeaways

- User input must never be trusted, whether it arrives through a URL parameter, a form, or a header.
- Encoding must match the context where the input lands (HTML body, attribute, JavaScript, URL).
- Basic method for finding XSS: send a unique string, locate it in the page source, identify the context it lands in, then craft a payload suited to that context.
- Understanding query strings and parameters is a fundamental building block of web security.




----------------------------------------------------------

# Reflected XSS into HTML context with nothing encoded

| বিষয় | তথ্য |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Category | Cross-Site Scripting (Reflected XSS) |
| Difficulty | Apprentice |
| Goal | `alert` ফাংশন কল করানো |
| Tools | Browser (Burp Suite ছাড়াই সমাধান সম্ভব) |

---

## ১. XSS কী?

XSS-এর পূর্ণরূপ Cross-Site Scripting। এটি এমন একটি দুর্বলতা, যেখানে আক্রমণকারী একটি ওয়েবসাইটের পেজে নিজের লেখা JavaScript ঢুকিয়ে দিতে পারে এবং সেই কোড victim-এর ব্রাউজারে ওয়েবসাইটটির নিজের কোড হিসেবেই চলে।

নামের ইতিহাস: প্রথমে এর নাম ছিল CSS (Cross-Site Scripting)। কিন্তু Cascading Style Sheets-এর সংক্ষিপ্ত রূপও CSS হওয়ায় বিভ্রান্তি এড়াতে এর সংক্ষিপ্ত রূপ করা হয় XSS। "Cross-site" শব্দটি এসেছে এই কারণে যে, আক্রমণ অন্য একটি উৎস (attacker-এর পাঠানো লিংক) থেকে এসে ভুক্তভোগী সাইটের ভেতরে কাজ করে।

XSS প্রধানত তিন ধরনের:

| ধরন | বৈশিষ্ট্য |
|---|---|
| Reflected XSS | পেলোড request-এ থাকে এবং সঙ্গে সঙ্গে response-এ প্রতিফলিত হয়। কোথাও জমা থাকে না। |
| Stored XSS | পেলোড সার্ভারে (যেমন database-এ) জমা থাকে এবং পেজ দেখা প্রতিটি ইউজারের উপর কাজ করে। |
| DOM-based XSS | সমস্যাটি সার্ভারে নয়, ক্লায়েন্ট-সাইড JavaScript-এর ভেতরে থাকে। |

এই ল্যাবটি প্রথম ধরন, অর্থাৎ Reflected XSS-এর সবচেয়ে সরল উদাহরণ।

---

## ২. ল্যাবের বর্ণনা

ল্যাবের ওয়েবসাইটে একটি search ফিচার আছে। সার্চ বক্সে যা লেখা হয়, সার্ভার তা কোনো ফিল্টার বা encoding ছাড়াই response পেজে বসিয়ে দেয়। ল্যাব সমাধানের শর্ত হলো এমন একটি পেলোড পাঠানো, যাতে পেজ লোড হওয়ার সময় `alert` ফাংশন চলে।

---

## ৩. Recon (পর্যবেক্ষণ)

### ধাপ ১: সার্চ বক্সে সাধারণ শব্দ দিয়ে পরীক্ষা

সার্চ বক্সে একটি নির্দিষ্ট ও সহজে চেনা শব্দ (`test123`) লিখে সার্চ করা হলো। এমন unique শব্দ ব্যবহারের কারণ হলো, পরে page source-এ `Ctrl+F` দিয়ে শব্দটি সহজে খুঁজে পাওয়া যায়।

### ধাপ ২: URL-এর পরিবর্তন লক্ষ্য করা

সার্চের পর ব্রাউজারের URL বদলে গেল:

```
https://LAB-ID.web-security-academy.net/?search=test123
```

URL-এর এই অংশগুলো আলাদা করে বোঝা দরকার:

| অংশ | অর্থ |
|---|---|
| `?` | এখান থেকে query string শুরু হয় |
| `search` | parameter-এর নাম |
| `=` | নাম ও মানের মাঝের বিভাজক |
| `test123` | parameter-এর value, অর্থাৎ ইউজারের লেখা ইনপুট |

এ থেকে বোঝা গেল, সার্চ বক্সের ইনপুট GET request-এর মাধ্যমে `search` নামের parameter হয়ে সার্ভারে যাচ্ছে। এই কারণে URL-এর মান বদলালেই ইনপুট বদলে যায়। সার্চ বক্সে না লিখে সরাসরি URL-এও পেলোড দেওয়া যায়, আর এটিই Reflected XSS-কে লিংকের মাধ্যমে ছড়ানোর উপযোগী করে তোলে।

### ধাপ ৩: Page source পরীক্ষা

পেজে right-click করে View Page Source (বা `Ctrl+U`) খুলে `test123` খোঁজা হলো। দেখা গেল শব্দটি HTML-এর একটি `<p>` ট্যাগের ভেতরে বসেছে:

```html
<p>test123</p>
```

(এটি সরলীকৃত রূপ, আসল পেজে ট্যাগের আশেপাশে আরও লেখা থাকতে পারে।)

### ধাপ ৪: Encoding আছে কিনা যাচাই

HTML-এ `<` ও `>` বিশেষ চিহ্ন, কারণ এগুলো দিয়ে ট্যাগ শুরু ও শেষ হয়। নিরাপদ ওয়েবসাইট ইনপুটের এই চিহ্নগুলোকে `&lt;` ও `&gt;`-এ রূপান্তর (encode) করে। এতে ব্রাউজার এগুলোকে ট্যাগ না ভেবে সাধারণ লেখা হিসেবে দেখায়।

এই ল্যাবে ইনপুটে কোনো encoding হচ্ছে না, অর্থাৎ ইউজারের লেখা চিহ্নগুলো হুবহু HTML-এ বসছে। এর মানে, ইনপুটে HTML ট্যাগ লিখলে ব্রাউজার সেটিকে আসল ট্যাগ হিসেবে পার্স করবে।

(ঐচ্ছিক যাচাই: সার্চ বক্সে `<b>test</b>` দিলে লেখাটি bold হয়ে দেখা গেলে প্রমাণ হয় যে ইনপুট HTML হিসেবে প্রসেস হচ্ছে।)

---

## ৪. Exploitation (আক্রমণ)

### পেলোড

```html
<script>alert(1)</script>
```

এটি একটি সাধারণ proof-of-concept পেলোড। `<script>` ট্যাগ ব্রাউজারকে JavaScript চালাতে বলে, আর `alert(1)` একটি পপআপ দেখায়। `alert` ব্যবহারের কারণ হলো, এটি নিরীহ কিন্তু স্পষ্টভাবে প্রমাণ করে যে নিজের লেখা কোড চলছে।

### পেলোড পাঠানো

সার্চ বক্সে পেলোডটি লিখে সার্চ করা হলো। URL-টি দাঁড়াল:

```
https://LAB-ID.web-security-academy.net/?search=<script>alert(1)</script>
```

ব্রাউজার URL-এর বিশেষ চিহ্নগুলো স্বয়ংক্রিয়ভাবে percent-encode করে পাঠায়:

```
?search=%3Cscript%3Ealert(1)%3C/script%3E
```

এখানে `%3C` হলো `<` এবং `%3E` হলো `>`। সার্ভার এটি decode করে আসল মান পায়।

### ফলাফল

সার্ভার পেলোডটি `<p>` ট্যাগের ভেতরে বসিয়ে response পাঠাল। ফলে পেজের HTML এমন হলো:

```html
<p><script>alert(1)</script></p>
```

ব্রাউজার পেজটি পড়ার সময় `<script>` ট্যাগ পেয়ে ভেতরের কোড চালিয়ে দিল। `alert` পপআপ দেখা গেল এবং ল্যাবে "Congratulations, you solved the lab!" বার্তা এল।

---

## ৫. কেন কাজ করল?

কারণগুলো ধাপে ধাপে:

1. সার্ভার ইউজারের ইনপুটকে কোনো পরিশোধন ছাড়াই response-এর HTML-এ বসিয়েছে।
2. ইনপুটের `<` ও `>` চিহ্ন encode হয়নি।
3. ব্রাউজারের HTML parser বুঝতে পারে না কোন অংশ ডেভেলপার লিখেছে আর কোন অংশ ইউজার ঢুকিয়েছে। সে পুরো response-টিকে বিশ্বাসযোগ্য HTML ধরে পড়ে।
4. ইনপুটটি HTML-এর body অংশে (HTML context) বসেছে, যেখানে `<script>` ট্যাগ সরাসরি বৈধ। তাই অতিরিক্ত কোনো কৌশলের (যেমন quote ভেঙে বের হওয়া বা attribute escape করা) প্রয়োজন পড়েনি।
5. ব্রাউজার script-কে সেই ওয়েবসাইটের origin-এর অংশ হিসেবে চালায়। তাই এর কাছে ওই সাইটের cookie, session ও DOM-এর অ্যাক্সেস থাকে।

---

## ৬. বাস্তব ঝুঁকি (Impact)

`alert(1)` শুধু প্রমাণ। বাস্তব আক্রমণে একই পথে এসব করা সম্ভব:

- Session cookie চুরি করে victim-এর অ্যাকাউন্ট দখল (session hijacking)
- জাল login ফর্ম দেখিয়ে পাসওয়ার্ড হাতিয়ে নেওয়া (phishing)
- Keylogger বসিয়ে ইউজার যা টাইপ করছে তা চুরি
- Victim-এর হয়ে অনুরোধ পাঠানো (যেমন পাসওয়ার্ড বা ইমেইল বদল)

Reflected XSS-এ আক্রমণকারীকে victim-কে ওই বিশেষ লিংকে ক্লিক করাতে হয়, যেমন ইমেইল, মেসেজ বা সোশ্যাল মিডিয়ার মাধ্যমে। এটিই Stored XSS-এর চেয়ে এর প্রধান সীমাবদ্ধতা।

---

## ৭. প্রতিরোধ (Mitigation)

| ব্যবস্থা | ব্যাখ্যা |
|---|---|
| Output encoding | আউটপুটের context অনুযায়ী `<` কে `&lt;`, `>` কে `&gt;`, `"` কে `&quot;` ইত্যাদিতে রূপান্তর করা। এটিই সবচেয়ে গুরুত্বপূর্ণ প্রতিরোধ। |
| Content Security Policy (CSP) | কোন উৎস থেকে script চলতে পারবে তা সীমিত করা। এতে inject করা inline script আটকে যায়। |
| HttpOnly cookie | JavaScript থেকে session cookie পড়া বন্ধ করা, ফলে cookie চুরি কঠিন হয়। |
| Input validation | প্রত্যাশিত ফরম্যাটের বাইরের ইনপুট বাতিল করা। তবে এটি একা যথেষ্ট নয়, encoding-এর পরিপূরক। |
| নিরাপদ framework | আধুনিক framework (যেমন React) ডিফল্টভাবে আউটপুট escape করে। |

---

## ৮. মূল শিক্ষা (Takeaway)

- ইউজারের ইনপুট কখনোই বিশ্বাসযোগ্য নয়, তা URL parameter, form বা header যেখান থেকেই আসুক।
- ইনপুট যেখানে বসছে (HTML body, attribute, JavaScript, URL), সেই context অনুযায়ী encoding করতে হয়।
- XSS খোঁজার প্রাথমিক পদ্ধতি: unique শব্দ পাঠানো, page source-এ তার অবস্থান দেখা, কোন context-এ বসছে বোঝা, তারপর সেই context-এর উপযোগী পেলোড বানানো।
- Query string ও parameter বোঝা web security-এর মৌলিক ভিত্তি।





