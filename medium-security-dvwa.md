## Medium security

### Reverse Shell in DVWA

We will use the same approach as we used for the low security case. We will need to somehow upload a reverse shell script in order to gain a remote access to the attacked machine. However, since the security level is higher, we will need to find more innovative way to do that.

I have tried a couple of approaches to bypass the validation. First, tried to upload a normal `.jpg` file to verify the correct file type.

![Screenshot 2022-03-12 004004](https://user-images.githubusercontent.com/19424915/158017947-958faff9-00cc-481b-9754-2dd87bbc9bca.png)

Then tried to:

1. Upload a file with a couple of extensions - `php-reverse-shell.img.txt.php`;

2. Upload a file with some capital letters in the extension - `php-reverse-shell.PHp`;
 
3. Upload a file with `;` and `\` separators - `image.jpg;php-reverse-shell.php`, `image.jpg\php-reverse-shell.php`.

None of the above approaches was successful.

Finally, I attempted to override the `Content-Type` of the `POST` http request which is resposible for uploading the file. To acheive that I had to use `Burp`.

First, I intersepted the `POST` request which uplaods the `php-reverse-shell.php` file.

![Screenshot 2022-03-12 023514](https://user-images.githubusercontent.com/19424915/158019691-e0ac683c-c657-408b-845f-fdbaef2ba8c5.png)

As we can observe, the `Content-Type` of the request body is `application/x-php`. I had to override it with `image/jpeg`. To acheive that, sent the intersepted request to the `repeater` and edited the `Content-Type` field.

![Screenshot 2022-03-12 023607](https://user-images.githubusercontent.com/19424915/158019809-d6d25cc9-7c89-4d01-acde-7aa2e413a9c9.png)

Making a request with the new `Content-Type` leads to a successful upload of the file to the server. Which means that the server validates only the `Content-Type` of the http request.

![Screenshot 2022-03-12 023637](https://user-images.githubusercontent.com/19424915/158019928-82c02d9b-6d4f-4caa-98d0-9ae6ff541307.png)

Now, we can navigate to the file and gain a remote access to the DVWA. Of course, we need to have a `netcat` listener running. 

![Screenshot 2022-03-12 023728](https://user-images.githubusercontent.com/19424915/158020050-baff31f0-db8d-438a-b62d-5a6f4e3dce69.png)

### Escalate Privilege

**Note: all the steps bellow are identical with those for the low security case, so will not be explained in details.**

Now, we can try to gain `root` user privileges. We will do the same steps as we did for the `low security` case.

First, we will upload `linux-exploit-suggester.sh` by serving it from our machine and downloading it from the attacked machine.

![Screenshot 2022-03-12 165735](https://user-images.githubusercontent.com/19424915/158027778-871b618f-8543-422f-85e0-c0a81307e671.png)

![Screenshot 2022-03-12 165759](https://user-images.githubusercontent.com/19424915/158027790-f69859e1-6d9d-4926-a2c0-3a49679848e5.png)

Then, we can execute the script.

![Screenshot 2022-03-12 165937](https://user-images.githubusercontent.com/19424915/158027822-3a7a20d5-13f4-4dcd-b85e-ad6a750fefde.png)

![Screenshot 2022-03-12 170027](https://user-images.githubusercontent.com/19424915/158027832-8a0ef4b8-08de-489d-94bc-8255c95377fe.png)

As we did for the low securty, we will use `rds` exploit.

We will transfer it to the attacked machine.

![Screenshot 2022-03-12 170124](https://user-images.githubusercontent.com/19424915/158028618-96556adb-539f-47ae-a609-c06ace48573c.png)
![Screenshot 2022-03-12 170202](https://user-images.githubusercontent.com/19424915/158028629-7a18ccb6-3101-436b-9aad-2ef70a33c0a8.png)

And execute it.

![Screenshot 2022-03-12 170252](https://user-images.githubusercontent.com/19424915/158028648-f16627bd-581c-4ff2-921d-8958183de62c.png)

![Screenshot 2022-03-12 170323](https://user-images.githubusercontent.com/19424915/158028652-a1820593-7de0-4555-8160-9bca0c7ecbce.png)
 
We have `root` access to the attacked machine when the security is set to **medium**.
