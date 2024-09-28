---
layout: post
title: Bypassing SSL Pinning in Flutter Based Android Applications
date: 2024-08-15
categories: [Android, writeup]
tags: [Android.SSL Pinning Bypass] 
---

### What is SSL pinning?    

SSL Pinning is a client-side technique used to prevent man-in-the-middle attacks by validating server certificates. A list of trusted certificates is embedded into the client application. At runtime, these certificates are compared against the server's certificates. If there is a mismatch, the connection is immediately disrupted, and no user data is transmitted to the server. This ensures that user devices communicate only with trusted servers.

By preventing man-in-the-middle attacks, SSL Pinning stops attackers from intercepting and modifying traffic. This, in turn, helps to protect against various server-side vulnerabilities, as attackers are unable to perform API-level tests. Thus, implementing SSL Pinning is essential for maintaining the security of your application.

&nbsp;

### Why is SSL pinning different in Flutter?

Flutter applications, which use the Dart programming language, do not utilize the system's CA (Certificate Authority) store. Instead, they use a list of CAs compiled directly into the application.

Regarding proxy settings, Dart applications are not inherently proxy aware. For instance, on Android, the HttpClient in Dart's dart library relies on proxy settings from environment variables like **http_proxy** and **https_proxy**. These settings are inherited from the Zygote process at system
startup, so any changes made to proxy settings after the device boots are not recognized by Dart applications.

Therefore, traditional methods for intercepting network traffic, such as modifying proxy settings or installing trusted certificates, are ineffective for Flutter applications on both Android and iOS.


Here, I’m explaining two methods for bypassing SSL pinning on a Flutter-based Android application:

&nbsp;

### Method 1: Using reFlutter

[reFlutter](https://github.com/Impact-I/reFlutter) is a Flutter reverse engineering framework that reverse engineers Flutter apps using a patched version of the Flutter library. This modified library bypasses SSL and intercepts traffic in Burp Suite

**Installation-**

`$ pip3 install reflutter==0.7.8`

**Usage-**

1. Execute the reFlutter command using the android apk.

 `$ reflutter app.apk`

 ![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image.png)


2. Select option 1 and enter the IP address of your Burp Suite Proxy Server.
3. Proceed to sign the resulting APK using [uber-apk-signer](https://github.com/patrickfav/uber-apk-signer)

 `$ java -jar uber-apk-signer.jar --allowResign -a release.RE.apk`

 ![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%201.png)


4. To intercept traffic, follow these steps in Burp Suite:
 - Go to the **Proxy** tab and navigate to **Options** under **Listener Proxy**.
 - Set the port to 8083.
 - Choose **All interfaces** for the **Bind to address** option.
 - Enable **Support invisible proxying**.
5. Open the application and observe the traffic captured in the Burpsuite.

No need to install any certificates, and root access isn't required on Android devices. reFlutter also provides a way to bypass certain Flutter certificate pinning implementations.

&nbsp;

### Method 2: Using Frida and ProxyDroid

To proxy the traffic, we can use iptables to route all traffic from the device to our proxy. On a rooted Android device, we can use the ProxyDroid app to create rules with iptables, directing all traffic from the device to the Burp Suite proxy.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%202.png)

&nbsp;

Using ProxyDroid, we can route traffic to Burp Suite; however, requests will fail due to SSL pinning. 

To bypass SSL pinning, we need to unpack the application's APK and reverse engineer the **libflutter.so** file.

To unpack the application APK use apktool, run the below command-

`$ apktool d app.apk`

Then, navigate to the **/lib/** directory where you'll see directories compiled for different platforms. Since our device is **arm64**, select **arm64-v8a**. and, open **libflutter.so** in the Ghidra reverse engineering tool.

The Flutter Engine relies on the BoringSSL library for SSL/TLS implementation. You can access the source code for BoringSSL here: [BoringSSL](https://github.com/google/boringssl).

To bypass SSL, we need to inspect **ssl_crypto_x509_session_verify_cert_chain** function, defined in **boringssl/ssl/ssl_x509.cc**. This function must be modified to always return true to bypass the check. We utilize a Frida script to dynamically accomplish this. But before that, we need to find the address of the **ssl_crypto_x509_session_verify_cert_chain** function in the disassembled code.

One way to locate it is by searching for the string **ssl_client** within the disassembled code, which corresponds to what is mentioned in the source code of BoringSSL.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%203.png)

&nbsp;

In Ghidra, we can search for the string **ssl_client** and then click on the location of the first result from the search outcome.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%204.png)

&nbsp;

The disassembler indicates that the analyzed libflutter.so has one cross reference (XREF) to the mentioned string. Clicking on the function will load the function in the decompiled screen.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%205.png)

&nbsp;

Examine the loaded function and observe that it includes two string: **ssl_client** and **ssl_server**. This confirms that the function we loaded corresponds to **ssl_crypto_x509_session_verify_cert_chain**. To further verify this, analyze the parameters defined in the disassembled code and note that both the source code and disassembled code have three arguments.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%206.png)

&nbsp;

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%207.png)


Note the address of the function; in my case, it's **7d1c90**. However, this isn't the actual address. In Ghidra, the base address starts from 100000. Therefore, we need to subtract 100000 from **7d1c90** which gives the actual address of **6dc730**.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%208.png)

&nbsp;

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%209.png)


To dynamically return true, you can use the script available at the following link:
[https://github.com/fatalSec/flutter_reversing/blob/main/flutter_ssl_bypass.js](https://github.com/fatalSec/flutter_reversing/blob/main/flutter_ssl_bypass.js)

Remember to replace your address at line 21 of the provided code. Now run the script and access the application. You will be able to intercept the traffic in Burp Suite.

![image.png](/assets/img/Blog_images/flutter-ssl-pinning-bypass/image%2010.png)





