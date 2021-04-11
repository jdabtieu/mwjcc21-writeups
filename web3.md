# The Cookie Jar
![](https://img.shields.io/badge/category-web-blue)

## Description
People love cookies. That’s why the Cookie Jar, a cookie-production factory, even exists in the first place. They make a large assortment of cookies, from chocolate chip cookies, to butter cookies, to browse- I mean what?

[http://138.197.69.9:30002](http://138.197.69.9:30002)

## Solution
The problem statements hints at looking at the browser cookies. By using the DevTools in most browsers, cookies will be available in the Application tab. In this problem, we see that a cookie called `disallowed` was created with the value `/privateemails`. Because of that slash, it seems like `/privateemails` is a page on this website. Navigating to the page however, we get a simple message: `Unauthorized`. Perhaps we need to remove the disallowed cookie to be able to access this page.

```
From: 'admin' <admin@mgcictf.club>
To: 'jdabtieu' <jdabtieu@mgcictf.club>
Subject: 'login'
Body:
    Hey jdabtieu, I noticed that your password is very weak so I changed it to [REDACTED]. You can still login from the same place, /login-60b8a008-0558-492a-a1aa-5084a8f95d6c.

    Sincerely,
    admin
```

Looks like we found a private email and another url to go to! The login page looks like a generic login page, however, the source code seems to suggest that there are no correct credentials.
```js
    $("form").submit(function (event) {
        event.preventDefault();
        alert("Wrong credentials lmao");
    });
```

Instead, if we look at the cookies, another cookie was created called auth! Since it contains only numbers and the letters a-f, it suggests that this is hex-encoded. Decoding it results in `{username: 'none', authenticated: false}`.

We should be able to just change authenticated to true and replace the cookie to gain access. However, replacing the new cookie (`7b757365726e616d653a20276e6f6e65272c2061757468656e746963617465643a20747275657d`) and refreshing the page gives an error that "That user doesn't exist"...

You could try bruteforcing usernames...or realize that two usernames were leaked in the email, specifically `admin` and `jdabtieu`.

First, we can try using admin as the username. It doesn't work, saying that admins don't log in here. However, replacing the username with jdabtieu does work, and we get in.

`Per company policy, you should be using Internet Explorer 6. You are using: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.90 Safari/537.36 Edg/89.0.774.57`

This seems to want us to change our user-agent to Internet Explorer 6. The user-agent is a string that identifies your browser to a website, and can be changed to anything you want. Changing my user-agent to `Internet Explorer 6` doesn't actually work though, and replaces the message with `Per company policy, you should be using Internet Explorer 6. You are using: Internet Explorer 6`. If you Google Internet Explorer 6's user-agent, the first result points to `Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)`. Using this as the user agent does work, and see a generic input panel.

Viewing the control panel's source code though, we see that there's a hidden link to `/flag`.

Just like before, trying to navigate to the page results in an unauthorized message. There is also an identity flag, which seemes like a base64 encoded string. Decoding it results in `{username: 'jdabtieu'}`. The page says that only admins can view the flag, so replacing jdabtieu with admin would likely give us the flag.

This time, we get a different message. `No 'flag' cookie detected. Detected cookies are: 'auth', 'identity'`. It appears that we need to create a `flag` cookie first. Creating a cookie named `flag` and with a blank value, the value of the cookie is overwritten with the flag.

Flag: `CTF{d31ici0us_c00ki3s}`
