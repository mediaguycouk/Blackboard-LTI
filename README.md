# Blackboard-LTI
While trying to learn LTIs into Blackboard, I'm trying to note things I've learnt, things I'm yet to understand (but know I need to) and resources that will hopefully be the reason I understand them in the future.

I've started with the DevCon2020 sessions. 

### Building a Very Simple REST API Web App from Scratch

https://devcon.blackboard.com/ultra/courses/_17_1/outline

This is in python, which I don't know too much about. Plenty of people have advised using BBRest - https://github.com/mdeakyne/BbRest - which was included in https://github.com/mark-b-kauffman 's devcon video above. 

Using this I've managed to use LTI 1.1 (two legged auth) to act as an administrator to find a user, find a uses's courses, find assignments in a course, download all completed results from an assignment swapping the Blackboard student ID with their University student ID.

The "trouble" with this is that I don't really understand why everything is working. The talks also talk about the importance of three legged auth where possible. I wouldn't have to find a user's courses if I could just use their own account to say "find my courses" for example. You immediately stop a student from being able to extract results as well.

### Migrate Your B2 to LTI Advantage and REST 

https://devcon.blackboard.com/ultra/courses/_14_1/outline

This gets me much closer to understanding LTI. I clearly understand what needs to be done, how the authentication works and how Blackboard handles their keys. I hadn't heard of a JWT before this course (although I had seen one before as they are what is swapped from Azure Single Sign On to Blackboard/CampusM). I don't yet understand how I'll be making them and how many of them are mine / the universities / Blackboards / just make them up.

### LTI Advantage platform

https://github.com/andyfmiller/LtiAdvantagePlatform

This is the project that I want to be working from as it's .net Core (although 2.2 and compiling as 3.1 has too many errors to be a quick fix). The downloaded project did seem to require https://github.com/LtiLibrary/LtiAdvantage/tree/master/src to be downloaded and added to the project. I then had to remove the project dependencies and re-add them from the solution as they were hardcoded into a different location.

The tool - https://github.com/andyfmiller/LtiAdvantageTool - is really where the magic lies. I don't understand what it is doing, although I can currently make it work on the LTI advantage platform, but not on Blackboard. I'm sure that the reason for this is that the instructions - https://andyfmiller.com/2018/12/28/launching-an-lti-1-3-resource-link-using-openid-connect-third-party-login/ - point you towards some generated public/private keys - https://lti-ri.imsglobal.org/keygen/index - that can be used in the app (and work), but Blackboard asks for a JWKS URL https://auth0.com/docs/tokens/concepts/jwks, which I haven't used before and I don't think I'm using the right keys. 

---

Found a tool to correctly turn a PEM into a JWKS - https://8gwifi.org/jwkconvertfunctions.jsp - however I still get the same errors

```
fail: AdvantageTool.Pages.OidcLoginModel[0]
      Invalid target_link_uri [https://[snip].ngrok.io/Tool/443f3a95b5f4e41e].
      
      
Error.
An error occurred while processing your request.

Request ID:
    |96bf5dd9be18cf43b58b499f75e825bb.eefedffa_
HTTP Status Code
    400
```

The link seems to work in Advantage Platform and does appear to match the launch URL running on localhost (via ngrok) `Launch URL  
https://localhost:44308/Tool/443f3a95b5f4e41e`

---

Looks like it's ngrok 
```
if (targetLinkUri.Host != Request.Host.Host) // [snip.ngrok.io] != [localhost]
  _logger.LogError($"Invalid target_link_uri [{TargetLinkUri}].");
```

---

Made OidcLogin have `if (targetLinkUri.Host != Request.Host.Host && !targetLinkUri.Host.EndsWith("ngrok.io"))`. That'll do for now and will need to be taken out.

That gets me past the first step. I'm now into the second leg as the app is directing me back to Blackboard, but with a `Could not find configuration for client_id: 9b8676f5-e88d-405f-a63e-4f83fdfc9e9e` error.

---

Another step forward. The error above was not understanding which GUID needed to be in the client ID in the tool settings Client ID field. The ID is not the developer application key or application ID. The ID is from https://developer.blackboard.com/portal/applications and should match the Client ID that has been added to Blackboard > System Admin > Integrations > LTI Tool Integration > Register LTI 1.3/Advantage Tool.

New error is `Invalid redirect_url: https://[snip].ngrok.io/Tool/443f3a95b5f4e41e`. Hopefully this one should be easier as it doesn't match the URL that the tool thinks it is on. I'm thinking it might be easier to tell visual studio to use the external hostname, but I'm not too sure how.

---

Blackboard seems to suggest that the redirect URL needs to be on the same hostname `The Tool Provider URL must be located on one of the configured host names.`, however in testing the Target Link URI needs to be exactly the same as `Tool Redirect URLs` in the LTI 1.3 settings. The plural seems to suggest that this could be a list, and that list is on https://developer.blackboard.com/portal/applications/edit, but also the Blackboard page doesn't seem to refresh so it is presumably difficult to add new items to this list. Certainly the domain whitelists only updated once I deleted the LTI and re-added it.

My LTI now loads, but with a message `No connection could be made because the target machine actively refused it`, which is within the try/catch of `// Using the JwtSecurityTokenHandler.ValidateToken method, validate four things:`. I guess that might still mean my JWT token isn't correct yet, but I'll go line by line through the debugger to find that out next.

---

Nope, I had two platforms registered, one was localhost:44308 and the other was [snip].ngrok.io. I hadn't updated Blackboard with the tool ID that referenced the ngrok address (and both localhost and ngrok.io were going to the same computer).

I now have a working LTI tool!

![alt text](https://github.com/mediaguycouk/Blackboard-LTI/blob/master/ltiss1.png "Resource link request details")

Although that does say invalid jwt token, but I can't tell if that is specific to that gradebook section. There's always something.

---

I had been misunderstanding deep links. I couldn't understand why the deep link url didn't work if it wasn't exactly the same as the single deep link URL in the Tool Redirects URL. The thing I'd been missing was that when you say something is a deep link then there is only one URL, but when you actually place it in your course it shows you your own webpage so that you can store there where the deep link should go.

---

The other thing I stumbled across was the answer to my JWKS questions. When you create a new application in Blackboard you get a set of keys, ClientId|Secret|Application name. When you manage those keys you can hit the + and get new keys (once you have more than one you can delete them so that you can rotate keys regularly). What I personally missed was the "Generate new Private Key" button, which will generate a JWKS Public Keyset URL AND generate a Private Key that can be used to sign the web tokens. 
