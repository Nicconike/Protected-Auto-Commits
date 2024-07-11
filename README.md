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
	![GitHub App](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Github_App.png)

3. Name the App and add a Homepage URL
	![Register](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Register.png)

> [!Note]
> The GitHub App Name must be unique
>
> For Homepage URL, you can provide any URL since it will be for your account only.

4. Permissions
	1. Only Repo permissions are needed for contents `Access: Read and Write`
	2. Metadata read only permission is mandatory `Access: Read-only`
	3. If needed you can also add Actions permission `Access: Read-only`

#### Installation

1. Installation
	1. Select `Only on this Account`
	![Installation](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Installation.png)
	2. Install the app to the repository where you need to push commits to a protected branch

2. Environment Variables
	1. APP ID: After App creation, copy the app id from the General section and save it your respective repo's env vars/secrets
	2. Private Key: Create a private key which is required to sign access token requests as shown below. Learn more about Private Keys from [here](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app#generating-a-private-key).
	![Private Key](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Private_Key.png)

3. Protected Branches
	1. If not enabled already, then enable branch protection rules in your repository and add the newly created Github App to bypass these rules
	2. Also, please make sure that you are creating a branch ruleset instead of the legacy or classic branch protection rule. Because the bypass won't work with classic rule.
	![Rulesets](https://github.com/Nicconike/Protected-Auto-Commits/blob/master/assets/Rulesets.png)

#### Configuration

1. Create a yml file in workflow folder let's say it as `release.yml`. Now, add below action as the 1st step in the workflow
	```yml
	steps:
          - name: GitHub App Token
            uses: actions/create-github-app-token@v1
            id: app-token
            with:
                app-id: ${{ secrets.APP_ID }}
                private-key: ${{ secrets.APP_PRIVATE_KEY }}
	```

2. For checkout step, use the created app token
	```yml
	- name: Checkout Repo
            uses: actions/checkout@v4
            with:
                fetch-depth: 0
                token: ${{ steps.app-token.outputs.token }}
	```

3. Also, use this token in other steps as per your requirements
4. You are all set!

#### Examples

Here are few real time examples which I use it for my own repositories

1. [Steam Stats](https://github.com/Nicconike/Steam-Stats)
	Workflow file - [release.yml](https://github.com/Nicconike/Steam-Stats/blob/master/.github/workflows/release.yml#L25)
	```yml
	steps:
          - name: GitHub App Token
            uses: actions/create-github-app-token@v1
            id: app-token
            with:
                app-id: ${{ secrets.APP_ID }}
                private-key: ${{ secrets.APP_PRIVATE_KEY }}

          - name: Checkout Code
            uses: actions/checkout@v4
            with:
                fetch-depth: 0
                token: ${{ steps.app-token.outputs.token }}

          - name: Set up Python
            uses: actions/setup-python@v5
            with:
                python-version: '3.12'
                cache: "pip"

          - name: Cache Dependencies
            uses: actions/cache@v4
            with:
                path: ~/.cache/pip
                key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
                restore-keys: |
                    ${{ runner.os }}-pip-

          - name: Install Dependencies
            run: |
                python -m pip install --upgrade pip
                pip install python-semantic-release

          - name: Semantic Release
            id: github-release
            uses: python-semantic-release/python-semantic-release@v9.8.5
            with:
                github_token: ${{ steps.app-token.outputs.token }}
	```

2. [Steam Stats](https://github.com/Nicconike/Steam-Stats)
	Workflow file - [codeql.yml](https://github.com/Nicconike/Steam-Stats/blob/master/.github/workflows/codeql.yml#L53)
	```yml
	steps:
          - name: GitHub App Token
            uses: actions/create-github-app-token@v1
            id: app-token
            with:
                app-id: ${{ secrets.APP_ID }}
                private-key: ${{ secrets.APP_PRIVATE_KEY }}

          - name: Checkout Code
            uses: actions/checkout@v4
            with:
                token: ${{ steps.app-token.outputs.token }}

          - name: Set up Python
            uses: actions/setup-python@v5
            with:
                python-version: "3.x"
                cache: "pip"

          - name: Cache Dependencies
            uses: actions/cache@v4
            with:
                path: ~/.cache/pip
                key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
                restore-keys: |
                    ${{ runner.os }}-pip-

          - name: Run Pylint and Generate Badge
            id: run-pylint
            run: |
                python -m pip install --upgrade pip
                pip install pylint
                pylint_output=$(pylint api tests || true)
                echo "$pylint_output"

                score=$(echo "$pylint_output" | grep -oP 'Your code has been rated at \K[0-9]+\.[0-9]+' || echo "0.0")
                color="red"
                if (( $(echo "$score == 10" | bc -l) )); then
                    color="brightgreen"
                elif (( $(echo "$score >= 9" | bc -l) )); then
                    color="yellow"
                elif (( $(echo "$score >= 8" | bc -l) )); then
                    color="orange"
                elif (( $(echo "$score >= 6" | bc -l) )); then
                    color="red"
                fi

                badge="![Pylint](https://img.shields.io/badge/Pylint-$score-$color?logo=python)"
                echo "PYLINT_BADGE=$badge" >> $GITHUB_OUTPUT

          - name: Update README with Pylint Badge
            if: github.ref == 'refs/heads/master'
            env:
                GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
            run: |
                sed -i 's|!\[Pylint\](.*)|${{ steps.run-pylint.outputs.PYLINT_BADGE }}|' README.md
                git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"
                git config --global user.name "github-actions[bot]"
                git add README.md
                git diff --quiet && git diff --staged --quiet || (git commit -m "chore: Update Pylint Badge" && git push origin HEAD:master)
	```

3. [Goautomate](https://github.com/Nicconike/goautomate)
	Workflow file - [release.yml](https://github.com/Nicconike/goautomate/blob/master/.github/workflows/release.yml#L17)
	```yml
	steps:
          - name: GitHub App Token
            uses: actions/create-github-app-token@v1
            id: app-token
            with:
                app-id: ${{ secrets.APP_ID }}
                private-key: ${{ secrets.APP_PRIVATE_KEY }}

          - name: Checkout Repo
            uses: actions/checkout@v4
            with:
                fetch-depth: 0
                token: ${{ steps.app-token.outputs.token }}

          - name: Setup Go
            uses: actions/setup-go@v5
            with:
                go-version: "1.22.x"

          - name: Semantic Release
            uses: go-semantic-release/action@v1
            id: semantic
            with:
                github-token: ${{ steps.app-token.outputs.token }}
                changelog-file: CHANGELOG.md
                update-file: go.mod
                changelog-generator-opt: "emojis=true"
	```

4. Automated GitHub Releases Example
	1. [Goautomate](https://github.com/Nicconike/goautomate/releases)
	2. [Steam-Stats](https://github.com/Nicconike/Steam-Stats/releases)


## Thanks for Reading!
