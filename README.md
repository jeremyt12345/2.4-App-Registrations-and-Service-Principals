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
