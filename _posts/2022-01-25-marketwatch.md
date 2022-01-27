---
layout: post
title: MarketWatch
---
### Summary
Marketwatch is a Windows program which sends notifcations on new deals from local markets such as Kijiji so you can be the first to get them. Marketwatch also supports automated ad reposting and more features are being added on.


### Background/Purpose

I started repairing phones around the start of Grade Nine. I would look over local markets like Facebook Marketplace, Kijiji, or Craigslist, and see if I could find a phone I could buy, repair, and sell for a profit. This eventually expanded to electronics repair in general and flipping whatever I could find, but the majority of money I made still came from phone repair.

Finding a good deal before everyone else was a huge advantage as they would usually be gone quite fast. I realized that this can be automated using my programming knowledge. I could use an IPO (Input, Process, Output) approach to search for available listings on the market using Kijiji’s API, filter for new listings that have appeared since last search, and if they were listed for a good price, I’d make my program notify me through a sound and console output. I used several software engineering techniques such as Dependency Injection (DI), Object Oriented Programming (OOP), and Serialization / Deserialization. I’ve also expanded the project to add automatic listing reposting and a graphical user interface (still in development).

### Development tools, Frameworks, etc
This project was made in C#/.NET Core, with Visual Studio 2019 as my IDE.

## Code and Design snippets

### Algorithmic Implementations

The code for notifying the end user of new ads that have appeared on Kijiji follows an algorithm:
1. Fetch new ads through script
2. Filter ads (could be price filter for example, done in same step as fetch)
3. Select newly posted ads
4. Print ads to console and play a sound if there are any newly posted ads

This series of instructions is encapsulated in an asynchronous method that repeats infinitely until a CancellationToken is triggered. Here is the method:
```cs
public async Task Scout(SearchParameters parameters, TimeSpan scoutInterval,
                        Action<Listing> notify, CancellationToken token) {
    var lastQueryTime = DateTime.Now;
    while (true) {
        await Task.Delay(scoutInterval, token);
        if (cancellationToken.IsCancellationRequested) return;

        var latestSearch = SearchAds(parameters);
        //For each ad posted after last query, invoke a delegate
        foreach (Listing listing in latestSearch.Where(listing => listing.Date > startedSearchingTime))
            notify.Invoke(listing);

        lastQueryTime = DateTime.Now;
    }
}
```

Ad reposting follows this algorithim:
1. Get token through login or if saved from previous login
2. Choose ad to repost (inputted from user)
3. Download the ad locally
4. Send an HTTP DELETE request to Kjijij to remove the ad
5. Wait 3 minutes (to circumvent duplicate ad detection on Kijiji’s end)
6. Send HTTP POST request to post the ad



### API Implementations
There was a publicly available GitHub repository that allowed me to fetch ads from Kijiji. The repo is written in Javascript, so I couldn’t import the code directly into my application and run it. I chose to create a Javascript script which I could run through a terminal with any additional arguments it would need (i.e., query string) also supplied via command line (CLA). After completion, the script would produce a JSON string outputted through a file either indicating a successful request and the retrieved ads or a failed request and error information. To avoid polymorphic JSON (bad design), the string is prepended by either 0 (success) or 1(failure):

Successful request:
```javascript
0{
  "ads":[...]
}
```

Failed request:
```javascript
1{
  "exception": "403 error encountered"
}
```

My code then takes the JSON and serializes it to an object so I can work with the data:
```cs
public List<Listing> SearchAds(SearchParameters parameters) {
  string paramsCla = parameters.ToCla();
  string strCmdText = "queryKijiji.js " + paramsCla;

  //Start the script
  Process proc = new Process {
      StartInfo = {CreateNoWindow = true, FileName = "node.exe", Arguments = strCmdText}
   };
  proc.Start();
  proc.WaitForExit();

  //Read the data
  string data = File.ReadAllText(parameters.Output + ".json");
  string json = data.Substring(1);
  File.Delete(parameters.Output + ".json");
  
  //Return data if succesful, throw exception otherwise
  if (data.StartsWith("0")) {
      return JsonConvert.DeserializeObject<List<Listing>>(json, jsonSerializerSettings);
  }
  else {
      var error = JsonConvert.DeserializeObject<KijijiModuleError>(json, jsonSerializerSettings);
      throw new Exception(error.Message);
  }
}
```      
In hindsight, a more elegant approach could have been implemented using interprocess communication via pipe. This would allow for more direct dataflow between the two processes (the script, and my C# program).  

To add support for reposting and other features, I also wrote some of the API calls myself. An example using my login method:
```cs
public async Task<HttpResponseContainer<LoginResponseXml>> Login(string email, string password) {
    const string url = "https://mingle.kijiji.ca/api/users/login";

    //Making payload variable through form encoding
    var keyValues = new List<KeyValuePair<string, string>> {
        new KeyValuePair<string, string>("username", email),
        new KeyValuePair<string, string>("password", password),
        new KeyValuePair<string, string>("socialAutoRegistration", "false")
    };
    HttpContent payload = new FormUrlEncodedContent(keyValues);

    HttpRequestMessage httpRequest = new HttpRequestMessage(HttpMethod.Post, url);

    httpRequest.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/xml"));
    httpRequest.Headers.Add("x-ecg-ver", "1.67");
    httpRequest.Headers.Add("x-ecg-ab-test-group", "");
    httpRequest.Headers.AcceptLanguage.Add(new StringWithQualityHeaderValue("en-CA"));
    httpRequest.Headers.AcceptEncoding.Add(new StringWithQualityHeaderValue("utf-8"));
    httpRequest.Headers.UserAgent.Add(UserAgentProduct);
    httpRequest.Headers.UserAgent.Add(UserAgentComment);
    httpRequest.Content = payload;

    HttpResponseMessage httpResponse = await _client.SendAsync(httpRequest);

    if (!httpResponse.IsSuccessStatusCode)
        return new HttpResponseContainer<LoginResponseXml>(httpResponse, null);

    LoginResponseXml loginResponse = 
        LoginResponseXml.XMLDeserialize(await httpResponse.Content.ReadAsStringAsync());
    var httpResponseContainer = new HttpResponseContainer<LoginResponseXml>(httpResponse, loginResponse);

    return httpResponseContainer;
}
```
<sub> Kijiji's JSON implementation was messy to say the least, I opted to return API data in XML </sub>

