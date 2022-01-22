# [SQL Union](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

We can observe that clicking on a link for a given category leads to a new GET request to the server which sends the corresponding category using query param called **category**. Therefore, we can imagine the SQL request the server makes. It could be something like the following:

``SELECT FROM PRODUCTS WHERE category='example_category'``

Now, we can try exploiting the server by appending  ``'+union+select+null--+`` to the end of the url. This leads to a server error, so we can keep adding ``null`` into the url, until we guess the table column count. Once we have the correct number of **nulls**, the error disappers and we can access all the data from the corresponding SQL table.

The desired request is ``'+union+select+null,null,null--+``

# [SQL Union from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

We can use the same approach as in the previous task. The only difference will be the fact that we have to select some data from another database table.

Sending the following query param will give us the usernames and passwords we need:

``category='+union+select+username,password+from+users--+``

We receive a list of usernames and passwords which contains the desired ``administrator``'s credentials:

``
username: administrator,
password: ha9wahag68c5asob27tp
``

# [SQL Login bypasss](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

The application sends the username and the password to the server using the body of a POST requst. We can try sending **'** as one of the parameters in order to test if the server is SQL injection vulnarable. Luckly, it is. So, now we can imagine the SQL request the server makes to the database. It could be something like the following:

``SELECT username FROM users WHERE user='login form username field' AND password='login form password field'``

We do have the username which we would like to impersonate with. The only thing we need is to bypass the password. We have to force the ``WHERE`` statement to return ``true`` for our user - ``adnministrator``. The SQL request we would like to acheive is something like the following:

``SELECT username FROM users WHERE user='administrator' AND password='whatever' OR 1=1``

Based on all of the above being said we can try to send a request to the server with the following POST body:

``username=administrator&password='OR 1=1 -- ``

This is enough to log in as the ``administrator`` user.

# [SQL Hidden data](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

We can use the same approach as in the previous task to make the ``WHERE`` expression to return ``true``. So, we can try sending the following as a query:

``category='+OR+1=1+--+``

This will retrieve all the data we need.

# [Time based SQL injection](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)

This time we have to send an injection using the cooking in the request. So, we have to do something like the following:

``TrackingId:id' || pg_sleep(10) -- ``

However, we do not know the type and the version of the database, the ``sleep`` function is different in the different databases. We can try ``SELECT SLEEEP(10)``, ``WAITFOR DELAY '0:0:10'`` or ``SELECT pg_sleep(10)``. In our case the desired query is:

``TrackingId:id' || pg_sleep(10) -- ``

# [Simple OS Command injection](https://portswigger.net/web-security/os-command-injection/lab-simple)

Clicking on the **Check Stock** button for a product sends a POST request to the server. We can try manipulating one of the params by attaching the ``whoami`` command. So, we will need to send a request containing something like the following:

``productId=4&storeId=1|whoami``

# [Time delay OS Command Injection ](https://portswigger.net/web-security/os-command-injection/lab-blind-time-delays)