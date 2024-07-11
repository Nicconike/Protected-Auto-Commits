# Protected-Auto-Commits
If you want to automate the commits for a bot in a protected branch as well then below approach is the best possible option

## Direct Approach
The direct or straight-forward approach is to just create a GitHub App for your organization/account and use it within your account with a bot like `github-actions[bot]` which can use github app's token to commit automatically to a protected branch

If you are new to [GitHub Apps](https://docs.github.com/en/apps/overview), then please follow below steps to create a new GitHub App for your work and then use it in the way you want.

### GitHub App
A GitHub App is a type of integration that you can build to interact with and extend the functionality of GitHub. You can build a GitHub App to provide flexibility and reduce friction in your processes, without needing to sign in a user or create a service account.

#### Register a New GitHub App
First, Creating a GitHub App for your organization or account. Please follow the official documentation [here](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app) or if that seems confusing then please follow below steps:

1. Goto your organization/account's setting then click on developer settings as shown below
	![Settings](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Settings.png)
	![Developer Settings](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Developer_Settings.png)

2. Create New GitHub App
	![GitHub App](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/GitHub_App.png)

3. Name the App and add a Homepage URL
	![Register](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Register.png)

> [!Note]
> THe GitHub App Name must be unique
>
> Also, you can provide any URL for since it will be for your account only
