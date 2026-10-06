# Stored XSS into HTML context with nothing encoded

| বিষয় | তথ্য 
|---|---|
| Platform | PortSwigger Web Security Academy |
| Category | Cross-Site Scripting (Stored XSS) |
| Difficulty | Apprentice |
| Goal | Comment-এর মাধ্যমে `alert` ফাংশন কল করানো |
| Tools | Browser (Burp Suite ছাড়াই সমাধান সম্ভব) |

---

## ১. Stored XSS কী?

Stored XSS-এ আক্রমণকারীর পেলোড সার্ভারে সংরক্ষিত (stored) হয়ে যায়, সাধারণত database-এ। পরবর্তীতে যে-ই সেই পেজ খোলে, সার্ভার সংরক্ষিত পেলোডটি পেজের সঙ্গে পাঠিয়ে দেয় এবং ভিজিটরের ব্রাউজারে সেটি চলে।

এই ধরনকে Persistent XSS-ও বলা হয়, কারণ পেলোড একবার জমা হলে মুছে না ফেলা পর্যন্ত থেকে যায়। "Stored" শব্দের অর্থ "জমা করা", আর "persistent" মানে "স্থায়ী"। দুটো নামই একই বৈশিষ্ট্য বোঝায়।

### Reflected XSS-এর সঙ্গে পার্থক্য

| বিষয় | Reflected XSS | Stored XSS |
|---|---|---|
| পেলোড কোথায় থাকে | শুধু request-এ (URL-এ) | সার্ভারের database-এ |
| Victim-কে কী করতে হয় | আক্রমণকারীর পাঠানো লিংকে ক্লিক | শুধু স্বাভাবিকভাবে পেজটি খোলা |
| কতজন আক্রান্ত হয় | লিংকে ক্লিক করা ব্যক্তি | পেজ দেখা প্রত্যেকে |
| ঝুঁকি | মাঝারি | বেশি |

---

## ২. ল্যাবের বর্ণনা

ল্যাবের ওয়েবসাইটটি একটি blog। প্রতিটি blog post-এর নিচে comment করার সুবিধা আছে। ইউজারের লেখা comment কোনো encoding ছাড়াই database-এ জমা হয় এবং পরে পেজে দেখানো হয়। ল্যাব সমাধানের শর্ত হলো এমন একটি comment জমা দেওয়া, যাতে পোস্টটি খোলার সময় `alert` ফাংশন চলে।

---

## ৩. Recon (পর্যবেক্ষণ)

### ধাপ ১: Input নেওয়ার জায়গা খোঁজা

ল্যাবের blog থেকে যেকোনো একটি post খোলা হলো। পোস্টের নিচে একটি comment ফর্ম পাওয়া গেল, যার ঘরগুলো:

| ঘর | কাজ |
|---|---|
| Comment | মন্তব্যের মূল লেখা |
| Name | মন্তব্যকারীর নাম |
| Email | ইমেইল ঠিকানা |
| Website | ওয়েবসাইটের লিংক |

এই সবগুলোই ইউজারের ইনপুট, এবং সবগুলো সার্ভারে জমা হয়। অর্থাৎ সম্ভাব্য আক্রমণের জায়গা একাধিক।

### ধাপ ২: সাধারণ comment দিয়ে পরীক্ষা

প্রথমে একটি unique শব্দ (`test123`) comment হিসেবে জমা দেওয়া হলো। Name, Email ও Website-এ সঠিক ফরম্যাটের মান দিতে হয়, যেমন ইমেইলে `@` এবং Website-এ `http://` বা `https://`।

জমা দেওয়ার পর "Thank you for your comment!" বার্তা এল। post-এর পেজে ফিরে গিয়ে দেখা গেল comment-টি তালিকায় যুক্ত হয়েছে। এ থেকে বোঝা গেল, ইনপুট সার্ভারে সংরক্ষিত হচ্ছে এবং পেজ খোলার সময় আবার দেখানো হচ্ছে।

### ধাপ ৩: Page source পরীক্ষা

Page source-এ (`Ctrl+U`) `test123` খুঁজে দেখা গেল, comment-টি HTML-এর ভেতরে একটি ট্যাগে (যেমন `<p>`) বসেছে:

```html
<p>test123</p>
```

### ধাপ ৪: Encoding আছে কিনা যাচাই

HTML-এ `<` ও `>` চিহ্ন দিয়ে ট্যাগ তৈরি হয়। নিরাপদ সাইট এগুলোকে `&lt;` ও `&gt;`-এ encode করে। এই ল্যাবে comment-এ কোনো encoding হচ্ছে না, অর্থাৎ ইউজারের লেখা চিহ্নগুলো হুবহু HTML-এ বসছে। এর মানে, comment-এ HTML ট্যাগ লিখলে ব্রাউজার সেটিকে আসল ট্যাগ হিসেবে পার্স করবে।

---

## ৪. Exploitation (আক্রমণ)

### পেলোড

```html
<script>alert(1)</script>
```

### পেলোড জমা দেওয়া

Comment ফর্মে এভাবে পূরণ করা হলো:

| ঘর | মান |
|---|---|
| Comment | `<script>alert(1)</script>` |
| Name | যেকোনো নাম |
| Email | বৈধ ফরম্যাটের যেকোনো ইমেইল (যেমন `test@test.com`) |
| Website | `http://test.com` |

**Post Comment** চাপার পর ফর্মের তথ্য POST request হিসেবে সার্ভারে গেল এবং পেলোডটি database-এ জমা হলো।

### পেলোড চালু হওয়া

Comment জমা দেওয়ার পর "Back to blog" লিংকে ক্লিক করে post-এর পেজে ফেরা হলো। সার্ভার database থেকে সব comment তুলে HTML বানাল। ফলে পেজের HTML এমন হলো:

```html
<p><script>alert(1)</script></p>
```

ব্রাউজার পেজটি লোড করার সময় `<script>` ট্যাগ পেয়ে ভেতরের কোড চালিয়ে দিল। `alert` পপআপ দেখা গেল এবং ল্যাবে "Congratulations, you solved the lab!" বার্তা এল।

লক্ষণীয়: এরপর এই post-এর পেজ যতবার খোলা হবে, পপআপ ততবার আসবে, কারণ পেলোডটি database-এ রয়ে গেছে।

---

## ৫. কেন কাজ করল?

1. সার্ভার comment-এর ইনপুট কোনো পরিশোধন ছাড়াই database-এ জমা রেখেছে।
2. পেজ বানানোর সময় সেই সংরক্ষিত ডেটা encoding ছাড়াই HTML-এ বসিয়েছে।
3. ব্রাউজার বুঝতে পারে না কোন অংশ ডেভেলপারের লেখা আর কোনটি ইউজারের ঢোকানো। পুরো response সে বিশ্বাসযোগ্য HTML হিসেবে পড়ে।
4. ইনপুট HTML body-তে বসেছে, যেখানে `<script>` ট্যাগ সরাসরি বৈধ।
5. Script-টি ওয়েবসাইটের নিজের origin-এ চলে, তাই cookie, session ও DOM-এ তার অ্যাক্সেস থাকে।

মূল ভুলটি হলো: **ইনপুটের নিরাপত্তা শুধু জমা নেওয়ার সময় নয়, দেখানোর সময়ও নিশ্চিত করতে হয়।** ডেটা যত আগেই জমা হোক না কেন, আউটপুটের মুহূর্তে encoding না করলে ঝুঁকি থেকে যায়।

---

## ৬. বাস্তব ঝুঁকি (Impact)

Stored XSS Reflected XSS-এর চেয়ে বেশি বিপজ্জনক, কারণ victim-কে কোনো বিশেষ লিংকে ক্লিক করাতে হয় না। স্বাভাবিকভাবে পেজ খুললেই আক্রমণ হয়ে যায়। সম্ভাব্য ক্ষতি:

- পেজ দেখা প্রত্যেকের session cookie চুরি (বিশেষত admin ঢুকলে পুরো সাইট দখলের ঝুঁকি)
- জাল login ফর্ম দেখিয়ে পাসওয়ার্ড হাতিয়ে নেওয়া
- ব্যবহারকারীকে ক্ষতিকর সাইটে redirect করা
- Victim-এর হয়ে অনুরোধ পাঠানো
- এক পেজ থেকে অনেক মানুষের উপর স্বয়ংক্রিয়ভাবে ছড়িয়ে পড়া (worm-এর মতো)

---

## ৭. প্রতিরোধ (Mitigation)

| ব্যবস্থা | ব্যাখ্যা |
|---|---|
| Output encoding | Database থেকে তোলা ডেটা পেজে দেখানোর সময় context অনুযায়ী encode করা (`<` কে `&lt;`, `>` কে `&gt;`)। এটিই সবচেয়ে গুরুত্বপূর্ণ ব্যবস্থা। |
| Input validation | প্রত্যাশিত ফরম্যাটের বাইরের ইনপুট বাতিল করা। এটি encoding-এর পরিপূরক, বিকল্প নয়। |
| Content Security Policy (CSP) | কোন উৎস থেকে script চলবে তা সীমিত করা, ফলে inject করা inline script আটকে যায়। |
| HttpOnly cookie | JavaScript থেকে session cookie পড়া বন্ধ করা। |
| নিরাপদ framework | আধুনিক framework ডিফল্টভাবে আউটপুট escape করে। |
| Rich text-এর ক্ষেত্রে sanitizer | HTML অনুমতি দিতে হলে বিশ্বস্ত library (যেমন DOMPurify) দিয়ে শুধু নিরাপদ ট্যাগ রাখা। |

---

## ৮. মূল শিক্ষা (Takeaway)

- যেখানে ইউজারের ইনপুট জমা হয় (comment, profile, review, message), সেখানে Stored XSS-এর সম্ভাবনা থাকে।
- Stored XSS পরীক্ষার পদ্ধতি: unique শব্দ জমা দেওয়া, পেজ রিলোড করে দেখা কোথায় ও কোন context-এ দেখাচ্ছে, তারপর উপযোগী পেলোড জমা দেওয়া।
- ফর্মের সব ঘর (Name, Email, Website ইত্যাদি) আক্রমণের সম্ভাব্য জায়গা, শুধু মূল comment নয়।
- Reflected XSS-এ পেলোড URL-এ যায়, Stored XSS-এ যায় database-এ। তাই Stored XSS-এর প্রভাব বেশি এবং স্থায়ী


-------------------------------------------------------------------------------------------------------


# Stored XSS into HTML context with nothing encoded

| Item | Details |
|---|---|
| Platform | PortSwigger Web Security Academy |
| Category | Cross-Site Scripting (Stored XSS) |
| Difficulty | Apprentice |
| Goal | Make the browser call the `alert` function through a comment |
| Tools | Browser (solvable without Burp Suite) |

---

## 1. What is Stored XSS?

In Stored XSS, the attacker's payload is saved on the server, usually in a database. Later, whenever anyone opens the affected page, the server includes the saved payload in the response and it runs in the visitor's browser.

This type is also called Persistent XSS, because once the payload is saved it stays until it is deleted. "Stored" and "persistent" describe the same property.

### Difference from Reflected XSS

| Aspect | Reflected XSS | Stored XSS |
|---|---|---|
| Where the payload lives | Only in the request (URL) | In the server's database |
| What the victim must do | Click a link sent by the attacker | Simply open the page normally |
| Who is affected | The person who clicked the link | Everyone who views the page |
| Risk | Medium | High |

---

## 2. Lab Description

The lab website is a blog. Each blog post has a comment section below it. User comments are saved in the database without encoding and are later displayed on the page. To solve the lab, a comment must be submitted that causes the `alert` function to run when the post is opened.

---

## 3. Recon

### Step 1: Finding where input is accepted

A blog post was opened from the lab site. A comment form was found below the post with these fields:

| Field | Purpose |
|---|---|
| Comment | The main comment text |
| Name | The commenter's name |
| Email | Email address |
| Website | A website link |

All of these are user inputs and all are stored on the server, so there are multiple potential injection points.

### Step 2: Testing with a harmless comment

A unique string (`test123`) was submitted as a comment first. The Name, Email, and Website fields require valid formats, for example `@` in the email and `http://` or `https://` in the website.

After submitting, the message "Thank you for your comment!" appeared. Going back to the post page showed the comment added to the list. This proved that the input is saved on the server and displayed again whenever the page loads.

### Step 3: Inspecting the page source

Searching for `test123` in the page source (`Ctrl+U`) showed that the comment was placed inside an HTML tag, such as `<p>`:

```html
<p>test123</p>
```

### Step 4: Checking whether encoding is applied

In HTML, `<` and `>` form tags. A secure site encodes them into `&lt;` and `&gt;`. In this lab, comments are not encoded, meaning the characters typed by the user are placed into the HTML exactly as entered. So if an HTML tag is written in a comment, the browser will parse it as a real tag.

---

## 4. Exploitation

### Payload

```html
<script>alert(1)</script>
```

### Submitting the payload

The comment form was filled in as follows:

| Field | Value |
|---|---|
| Comment | `<script>alert(1)</script>` |
| Name | Any name |
| Email | Any validly formatted email (e.g. `test@test.com`) |
| Website | `http://test.com` |

After clicking **Post Comment**, the form data was sent to the server as a POST request and the payload was saved in the database.

### Payload execution

After submitting, the "Back to blog" link was clicked to return to the post page. The server fetched all comments from the database and built the HTML. The page HTML became:

```html
<p><script>alert(1)</script></p>
```

While loading the page, the browser encountered the `<script>` tag and executed the code inside. The `alert` popup appeared and the lab displayed "Congratulations, you solved the lab!"

Note: every time this post page is opened from now on, the popup will appear again, because the payload remains in the database.

---

## 5. Why Did It Work?

1. The server saved the comment input in the database without any sanitization.
2. When building the page, it inserted the stored data into the HTML without encoding.
3. The browser cannot tell which part was written by the developer and which was injected by a user. It reads the whole response as trusted HTML.
4. The input landed in the HTML body, where a `<script>` tag is directly valid.
5. The script runs within the website's own origin, so it has access to cookies, the session, and the DOM.

The core mistake: **input safety must be ensured not only when data is accepted, but also when it is displayed.** No matter how early the data was stored, failing to encode at the moment of output leaves the risk in place.

---

## 6. Real-World Impact

Stored XSS is more dangerous than Reflected XSS because the victim does not have to click any special link. Simply opening the page normally triggers the attack. Possible damage:

- Stealing session cookies of everyone who views the page (if an admin visits, the whole site may be compromised)
- Showing a fake login form to capture passwords
- Redirecting users to malicious sites
- Performing actions on behalf of the victim
- Spreading automatically from one page to many users (worm-like behavior)

---

## 7. Mitigation

| Measure | Explanation |
|---|---|
| Output encoding | Encode data according to context when displaying it from the database (`<` to `&lt;`, `>` to `&gt;`). This is the most important defense. |
| Input validation | Reject input that does not match the expected format. It complements encoding but does not replace it. |
| Content Security Policy (CSP) | Restrict which sources scripts may run from, blocking injected inline scripts. |
| HttpOnly cookies | Prevent JavaScript from reading session cookies. |
| Secure frameworks | Modern frameworks escape output by default. |
| Sanitizer for rich text | If HTML must be allowed, use a trusted library (such as DOMPurify) to keep only safe tags. |

---

## 8. Key Takeaways

- Stored XSS can exist anywhere user input is saved (comments, profiles, reviews, messages).
- Method for testing Stored XSS: submit a unique string, reload the page to see where and in what context it is displayed, then submit a suitable payload.
- Every form field (Name, Email, Website, etc.) is a potential injection point, not only the main comment.
- In Reflected XSS the payload travels in the URL, in Stored XSS it is saved in the database. This makes Stored XSS more impactful and persistent.

