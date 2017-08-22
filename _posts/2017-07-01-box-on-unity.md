---
title: Box API on Unity on Mac
teaser: Quick guide for using Box' API in Unity
category: problems
tags: [unity, box]
---

--------------------------------------

Forewords
--------------------------------------
Recently I was migrating Box's API to one of my iOS app using Unity on Mac.

Unity has a special edition for only mac. I originally tried to use its .NET SDK, but the SDK is not compatible with Unity's .NET subset. Please message me if you have a good solution to migrate its SDK to Unity and that would be helpful for everyone who is developing Unity games and need Box as a simple back-up facilities.

I used its REST API instead. Below would be a simple guide to use them.

Here I would only talk about steps for OAuth 2.0 with JWT authentication method.

Box Setup
-----------------------------------------

Follow Box's [instructions](https://developer.box.com/docs/configuring-box-platform) to setup an app under your account. Then generate public/private keys in .pem extension.

The autentication part is a bit confusing for people like me who are not familiar with encryption and OAuth 2.

I followed [this guide](https://developer.box.com/docs/authentication-with-jwt) to do so.

In the example request, it has several parts including the JWT assertion. The other parts are straightforward so I'm not going to go over them. 

JWT assertion is just an encryption of several json objects. It has three parts, header, claims and signiture. Notice that the signiture has to be base64URL encoded, not base64 encoded.

I looked through several SDKs for JWT encryption provided in the recommended in [this link](https://jwt.io/#libraries) in .NET. None of them is compatible with Unity... I ended up using a library [Chilkat](https://www.chilkatsoft.com/mono.asp). It has Mono version and is compatible with Unity. The following paragraph will tell you how to setup Chilkat.

Chilkat Installation
-----------------------------------------
Download its code from the website listed above and then include its chilkatCs folder in the Assets folder in the unity project. Then you need to include its dll into Assets too. You may notice that in nativeDll folder, the extension is .dylib. In Unity you can only include Mac package using .bundle extension.  Therefore you need to change libchilkatMono-9_5_0.dylib to libchilkatMono-9_5_0.bundle. Then you are able to go.

Sample Code
-----------------------------------------
Here is a sample code for getting access token. First time authorization code is also similar to this.

I also used another SDK Json.Net, [here](https://github.com/SaladLab/Json.Net.Unity3D/releases ) is the Unity compatible version.

Here are some points you need to watch out for.

First, the expiration time can only be a time less than 60 seconds. Other times would fail.

Second, JTI must be unique for every API request. I just use the timestamp as the unique jti.

<pre><code>
IEnumerator GetToken(System.Action<string> callBack) {
	// Unlock Chilkat SDK
	Chilkat.Global glob = new Chilkat.Global();
	bool success = glob.UnlockBundle("Anything for 30-day trial");
	if (success != true) {
		Debug.Log(glob.LastErrorText);
	}

	var secret = "secret"; // your own secret used to generate private key

	// load private key
	Chilkat.PrivateKey privKey = new Chilkat.PrivateKey();
	success = privKey.LoadEncryptedPemFile("/Users/InternAccount/Desktop/private_key.pem",secret);

	// JWT SDK
	Chilkat.Jwt jwt = new Chilkat.Jwt();

	//  Build the JWT header
	Chilkat.JsonObject jose = new Chilkat.JsonObject();
	//  Use RS256.  Pass the string "RS384" or "RS512" to use RSA with SHA-384 or SHA-512.
	success = jose.AppendString("alg","RS256");
	success = jose.AppendString("typ","JWT");
	success = jose.AppendString("kid",Public_Key_Id);

	// Build JWT claims
	Chilkat.JsonObject claims = new Chilkat.JsonObject();
	success = claims.AppendString("iss",Client_Id);
	success = claims.AppendString("sub",Sub);
	success = claims.AppendString("aud","https://api.box.com/oauth2/token");
	success = claims.AppendString("box_sub_type", Box_Sub_Type);
	int curDateTime = jwt.GenNumericDate(0);

	// Box only allows exp time to be 60 seconds; here I used recommended 30 seconds
	Debug.Log("time " + (curDateTime + 30));
	success = claims.AddIntAt(-1,"exp",curDateTime + 30); 
	// jti has to be unique for every request
	success = claims.AppendString("jti", UNIQUE_JTI + curDateTime.ToString());
	jwt.AutoCompact = true;

	// Generate JWT assertion
	string jwt_assertion = jwt.CreateJwtPk(jose.Emit(),claims.Emit(),privKey);
	Debug.Log ("assertion " + jwt_assertion);

	// Generate payload for Box API call
	Dictionary<string, string> content = new Dictionary<string, string>();
	content.Add("grant_type", "urn:ietf:params:oauth:grant-type:jwt-bearer");
	content.Add("client_id", Client_Id);
	content.Add("client_secret", Client_Secret);
	content.Add("assertion", jwt_assertion);

	UnityWebRequest www = UnityWebRequest.Post("https://api.box.com/oauth2/token", content);

	yield return www.Send();
	Token token = new Token();
	if (!www.isNetworkError) {
		string resultContent = www.downloadHandler.text;
		Debug.Log ("inside" + resultContent + "  " + fileName);
		token = JsonUtility.FromJson<Token> (resultContent);
		Debug.Log ("token" + token.access_token);
	}
	callBack (token.access_token);
}
</code></pre>

