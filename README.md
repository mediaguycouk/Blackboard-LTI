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
