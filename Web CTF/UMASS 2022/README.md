# UMASS CTF 2022 WEB - Autoflag
## Question
    Hmmmm. This website is kinda sus... Can you become the imposter and vent towards the flag?
http://34.148.103.218:4446
## Write-up
When you open the website,you'll see this
![image](https://user-images.githubusercontent.com/94618005/162439259-6341e608-303c-4c06-bb07-301107674747.png)
then if you click on **give me flag** button,and you'll get
![image](https://user-images.githubusercontent.com/94618005/162439568-5d1b9ae7-e85e-4786-b0ca-e1b51c535802.png)

and we back to the home page,if you click on **checkout the autoflag API**,you'll lead to a github website looks like this
![image](https://user-images.githubusercontent.com/94618005/162440367-62054cef-08a3-4626-a452-6e9e3e37a3e6.png)

So first,i trying to intercept every page that i can visit.
when i intercepted cheakout the autoflag API's request,you'll see this
![image](https://user-images.githubusercontent.com/94618005/162441325-cb5f8cb6-aac8-427d-88ec-18c8c2b235e1.png)
As you can see,the response told users do not change their identify from **anonymous** to **admin**, hmmmm... it's seems like a little sus!,  
and also told users check out their github cus they moved their server to server-side.

So we checkout the source code
#### Source Code
    //AUTOFLAG API V.1 : AUTOMATICALLY AUTHENTICATE USERS THEN REDIRECT TO FLAG
    function base64url(source) {
        encodedSource = btoa(source);
        while (encodedSource.endsWith('=')) {
            encodedSource = encodedSource.substring(0, encodedSource.length - 1)
        }
        encodedSource = encodeURI(encodedSource)
        console.log(encodedSource)
        return encodedSource;
    }

    function getSignedHMAC(unsignedToken) {
        return new Promise((resolve, reject) => {
            var xhr = new XMLHttpRequest()
            xhr.open("POST", 'http://34.148.103.218:4829/api/sign-hmac', true)

            //Send the proper header information along with the request
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")

            xhr.onreadystatechange = function () { // Call a function when the state changes.
                if (this.readyState === XMLHttpRequest.DONE && this.status === 200) {
                    resolve(xhr.responseText)
                }
            }
            xhr.send(`message=${unsignedToken}`)
        })
    }

    async function signToken() {
        header = `{"typ":"JWT","alg":"HS256"}`
        data = `{"fresh":false,"iat":1648755893,"jti":"a5139f01-3e20-446c-9627-16580f32f118","type":"access","sub":"anonymous","nbf":1648755893,"exp":1648756793}`
        unsignedToken = base64url(header) + "." + base64url(data)
        console.log(unsignedToken)
        let signature = await getSignedHMAC(unsignedToken)
        signature = signature.replaceAll('+', '-').replaceAll('=', '')
        let JWT = unsignedToken + "." + signature
        document.cookie = JWT
    }

    signToken()

    document.getElementById('redir').innerText = 'Redirecting you to your flag shortly'
    setTimeout(() => {
        window.location.replace("/flag");
    }, 2000)
   when you see the source code,you can find out that this problem will send a POST requests to **/api/sign-hmac** and get a unsigned Token,
   then use it to create a new JWT.
   So we just need grab the code,and paste it in the console of this challenge.and add a console.log() under JWT.
   don't forget try to change your identify from anonymous to admin.
   Then run it,and will get this back
   ![image](https://user-images.githubusercontent.com/94618005/162445164-b62ff40e-0374-48e2-96a7-a3f1b58ebdd8.png)
   
   So now we have a new JWT that the users identify is admin,and we changed cookies with this new JWT,then we send a GET requests to /flag,and you will get
   
  ![image](https://user-images.githubusercontent.com/94618005/162447392-f9a5aa35-9601-4739-80a3-f6a47ab72fa0.png)

   If get the response like "token is expired",don't be nervous,change your exp to another datetime before you print out the new JWT. 
  
