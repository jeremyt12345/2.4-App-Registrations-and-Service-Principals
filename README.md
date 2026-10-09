# 2.4-App-Registrations-and-Service-Principals
Reviewing Details on App Registrations


What I'm doing here is taking the work of the Mad Hat Security Program and writing it in my own words so I can better understand it.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Not every sign-in in a tenant is a human. Scripts, pipelines, and apps authenticate too — like a backup job or a web app pulling from storage — and they do it constantly, without anyone typing a password. These machine identities need to be locked down even more than human ones, because they’re often provisioned with privileged access by default


"When you register an application in Entra ID, you create an application object."

"The application object holds the global settings for the app: its name, its unique Application (client) ID, what it's allowed to ask for (permissions), and where its credentials live."

But blueprints cant sign in, this is where service principals come into the picture.

Service Principal the actual usable identity created from the blueprint. Actually signs-in, shows in sign-in logs, and what you assign roles/permissions.

<img width="1171" height="336" alt="image" src="https://github.com/user-attachments/assets/dcdb929f-58c1-4bf3-98ce-bf0fe06bbe64" />


The reason why having different kinds of accounts for organizations is helpful is that an app built for your tenant will be single tenant and one service principal. But an app used by multiple companies (like an SaaS app) will be multitenant. 

"Same Application (client) ID everywhere, a different service principal (and a different directory ID) in each tenant. That is the entire mechanism behind "sign in with your Microsoft work account" buttons all over the internet."


Authentication Via Code
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
We as users just tap our Microsoft MFA notifiaction to authenticate but obviously a computer cannot do that...yet.

A client secret is basically a password for your app. Its a string you generate on app registration and input in your script. This must always be secure in a secure place, anyone who has it can authtenticate as your app. Hence the "secret"


A certificate is more secure and what Microsoft recommends. It is a  cryptographic credential.


A managed identity is the safest because you dont run the risk of leaving the credential exposed anywhere. "Managed identity’s fix: Azure handles the whole credential lifecycle for you behind the scenes — creating it, rotating it, protecting it. You never see or touch the actual secret at all. You just tell a resource “use your managed identity” and Azure handles proving its identity internally. No secret for you to store, no secret to leak, nothing for an attacker to steal."


<img width="1158" height="297" alt="image" src="https://github.com/user-attachments/assets/40d50424-a403-476e-a226-9b7958d6dc8a" />

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

What an App Can Actually Do

An app authenticating is only half of what it can do and what you need to secure. After the authentication it still needs permissions to reach out to Microsoft Graph or your Azure Storage Account. The permissions would live in the app registration section. 


Delegated permissions
"Delegated permissions are the kind of permissions an app exercises/uses on behalf of a signed-in user. The user signs in, the app gets a token that represents itself as that user, and whatever the app does is limited to what that user could already do. If a delegated permission grants Mail.Read, the app can read the signed-in user's mail BUT not anyone else's."

"Think about Microsoft Teams or Outlook integrating with a third-party app — say, a scheduling tool like Calendly that connects to someone’s Outlook calendar.

When you (a user) go through that app’s “Connect your calendar” flow, you sign in with your Microsoft account and get a consent prompt saying something like “Calendly wants to: Read your calendar.” Once you approve, Calendly gets a token tied specifically to you. From that point on, Calendly can see your calendar and book meetings on your behalf — but it has zero visibility into anyone else’s calendar at EMS. If Josh or Daniel never went through that same consent flow, Calendly has no access to their calendars at all, even though it’s the exact same app."

Application Permissions
Are what it sounds like, the permissions are given to the app after it authenticates with the secret or cert.

These two are very different in regards to security. A leaked credential on an app with delegated permissions gives the attacker access with usually just one user. On the other hand a leaked credential with app permissions that also may have ".All" give the attacker range of the whole tenant as an admin.

<img width="1219" height="316" alt="image" src="https://github.com/user-attachments/assets/e7ee1f32-b9fd-4f70-aa4b-ea566ed2a868" />

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Where Tokens Land

After a user authenticates, Entra ID sends them back to the app at a specific, pre-registered web address — that address is called the Redirect URI. It’s where the token actually gets delivered. 

This next part I want to copy and paste straight from the source because the way it is explained is perfect and doesnt need to be simplfied or shortened:

"The flow goes like this. A user clicks Sign in inside some app. The app sends the user's browser to login.microsoftonline.com with the app's Application (client) ID and the redirect URI it expects to come back to. The user signs in there with their password and MFA. Microsoft generates an authorization code, a short-lived string that proves "this user just successfully authenticated for this app." Microsoft then sends the user's browser back to the redirect URI, with the authorization code attached to the URL. The app receives the user, takes the code, and exchanges it (along with its client secret) for an access token that lets it call APIs as that user. Again, this is delegated permissions.



Microsoft validates the redirect URI strictly. It must be an exact match against one of the URIs registered on the app registration. You cannot just ask Microsoft to send the code to any random URL. That validation is what stops random attackers from redirecting tokens to servers they control.



But here is where the attack begins: if somebody can register a malicious redirect URI on a legitimate app, the validation passes for that URI. Anyone with edit rights on the app reg can do this: the app's Owners, a Cloud Application Administrator, an Application Administrator, etc. If a phisher compromises an Owner's account, they add https://attacker.example/callback to the app's redirect URI list, then send victims a sign-in link using that app's client ID. When a victim clicks it and signs in, the authorization code goes straight to the attacker's server. If the attacker also has the client secret, they trade the code for tokens that act as the victim. The user thinks they signed in to a legit app, because well...they did. The tokens just landed somewhere they should not have."

<img width="1193" height="478" alt="image" src="https://github.com/user-attachments/assets/08cd7de4-76ab-4b56-bc1f-de0aa3f7cdf9" />

