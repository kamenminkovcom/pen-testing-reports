# [SQL Union](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

We can observe that clicking on a link for a given category leads to a new GET request to the server which sends the corresponding category using query param called **category**. Therefore, we can imagine the SQL request the server makes. It could be something like the following:

``SELECT FROM PRODUCTS WHERE category='example_category'``

Now, we can try exploiting the server by appending  ``'+union+select+null--+`` to the end of the url. This leads to a server error, so we can keep adding ``null`` into the url, until we guess the table column count. Once we have the correct number of **nulls**, the error disappers and we can access all the data from the corresponding SQL table.

The desired request is ``'+union+select+null,null,null--+``

![Screenshot from 2022-01-23 00-55-08](https://user-images.githubusercontent.com/19424915/150658016-e56f3c4a-b944-4574-a769-48e025b98889.png)

# [SQL Union from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

We can use the same approach as in the previous task. The only difference will be the fact that we have to select some data from another database table.

Sending the following query param will give us the usernames and passwords we need:

``category='+union+select+username,password+from+users--+``

We receive a list of usernames and passwords which contains the desired ``administrator``'s credentials:

``
username: administrator,
password: ha9wahag68c5asob27tp
``

![SQL-Union-from-other-tables](https://user-images.githubusercontent.com/19424915/150657622-288e3926-ef7b-4b15-9fba-5e44f49a78ba.png)


# [SQL Login bypass](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

The application sends the username and the password to the server using the body of a POST requst. We can try sending **'** as one of the parameters in order to test if the server is SQL injection vulnarable. Luckly, it is. So, now we can imagine the SQL request the server makes to the database. It could be something like the following:

``SELECT username FROM users WHERE user='login form username field' AND password='login form password field'``

We do have the username which we would like to impersonate with. The only thing we need is to bypass the password. We have to force the ``WHERE`` statement to return ``true`` for our user - ``adnministrator``. The SQL request we would like to acheive is something like the following:

``SELECT username FROM users WHERE user='administrator' AND password='whatever' OR 1=1``

Based on all of the above being said we can try to send a request to the server with the following POST body:

``username=administrator&password='OR 1=1 -- ``

This is enough to log in as the ``administrator`` user.

![Screenshot from 2022-01-23 00-48-52](https://user-images.githubusercontent.com/19424915/150658032-4a988f15-c976-42ca-a03a-dcf3d6758c27.png)

# [SQL Hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

We can use the same approach as in the previous task to make the ``WHERE`` expression to return ``true``. So, we can try sending the following as a query:

``category='+OR+1=1+--+``

This will retrieve all the data we need.

![Screenshot from 2022-01-23 00-51-31](https://user-images.githubusercontent.com/19424915/150658026-8fc294a5-21be-4817-80b3-38abea8dd48a.png)

# [Time based SQL injection](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)

This time we have to send an injection using the cookie in the request. So, we have to do something like the following:

``TrackingId:id' || pg_sleep(10) -- ``

However, we do not know the type and the version of the database, the ``sleep`` function differs in the different databases. We can try ``SELECT SLEEEP(10)``, ``WAITFOR DELAY '0:0:10'`` or ``SELECT pg_sleep(10)``. In our case the desired query is:

``TrackingId:id' || pg_sleep(10) -- ``

![time-based-sql](https://user-images.githubusercontent.com/19424915/150657655-4a828581-e8b2-418f-aff3-9e74b49205db.png)

# [Simple OS Command injection](https://portswigger.net/web-security/os-command-injection/lab-simple)

Clicking on the **Check Stock** button for a product sends a POST request to the server. We can try manipulating one of the params by attaching the ``whoami`` command. So, we will need to send a request containing something like the following:

``productId=4&storeId=1|whoami``

![whoami](https://user-images.githubusercontent.com/19424915/150657667-e24a7784-5e64-49c8-b962-190eeb57851d.png)

# [Time delay OS Command Injection](https://portswigger.net/web-security/os-command-injection/lab-blind-time-delays)

For this one we are dealing with a blind injection, meaning we will not receive a response feedback from the server, so we will have to somehow distinguish if our injection works or not. In our case, a request without a feedback is present when the "Feedback" form is sumbitted. We can try to append ``ping -c 10 localhost`` to every field and if we do not receive a response immediately, we will be sure that our injection works.

In our case the desired POST body is:
``name=name&email=name%40name.com||ping+-c+10+localhost||&subject=subject&message=something``

![time-delay-os](https://user-images.githubusercontent.com/19424915/150657683-6b4f074b-c873-4aa1-b7e7-7cfc659d5119.png)

# [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)

This is quite simple. We will just need to add, as a search term in the url, a script tag with some JavaScript code inside. Something like the following:

``search=<script>alert("pishi%20i%20bqgai")</script>``

# [Stored XSS](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded)

This time we send JavaScript code as a comment by using ``script`` tag, this will be sent to the server and once someone fetches the comment, the JavaScript code will be executed. In our case it should be something like the following:

``<script>alert('Alert!')</script>``

However, it could be much more dangerous, like a script which collects all the client data.

The POST request body for the comment in our case will be:

``postId=6&comment=%3Cscript%3Ealert%28%27something%27%29%3C%2Fscript%3E&name=name&email=name%40name.com&website=http%3A%2F%2Fname.com``

# [DOM based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink)

For this task we need to open the browser dev tools. When we do that, we should type something into the search box. We can observe that there is an ``img`` tag (because reasons) which has a ``src`` attribute with our search term in the end. For example, if we search for ``ss`` the ``img`` tag will look like: ``<img src="/resources/images/tracker.gif?searchTerms=ss">``. So, we can take advantage of that by placing another ``img`` tag with ``onerror`` event. In our case it could look like the following:

``"><img onerror="alert()" src=""/>``

![Screenshot from 2022-01-23 00-01-44](https://user-images.githubusercontent.com/19424915/150657718-a2c23c3e-8c68-470e-bf7f-ac481649a794.png)
