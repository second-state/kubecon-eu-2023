# Create an event driven microservice with ChatGPT and GitHub!

## What you will learn

* A serverless way to get a GitHub bot to review and summarize Pull Requests. It helps busy open source contributors understand and make decisions on PRs faster! 


## Prerequisites
* You will need an OpenAI API key. If you do not already have one, sign up [here](https://openai.com/blog/openai-api). Or the flows.network will provide you a free one for seven days.

* You will need to sign into flows.network from your GitHub account. It is free.

## Build and deploy your own GitHub Repo

Since there is a serverless way, we don't need to configure the environment locally.

### build and deploy the function

1 For the [github-pr-summary](https://github.com/flows-network/github-pr-summary) repo.

2 Go to flows.network and click on Create a flow button.

3 Import the `github-pr-summary` github repo you just forked

4 Click the Advanced test to see more settings. Here we need to set up enviornment. 

![image](https://user-images.githubusercontent.com/45785633/233407759-a1510561-abeb-485c-9c6d-a6b60581a0c0.png)

* login: Your personal GitHub id. The GitHub app will act as you when posting reviews.
* owner: GitHub org for the repo you want to deploy the 🤖 on.
* repo : GitHub repo you want to deploy the 🤖 on.
* trigger_phrase: The magic phrase to trigger a review from a PR comment.

After that, Click the deploy button to deploy the button.

### Configure the SaaS integrations

After that, flows.network will direct you to configure the external services required by your flow function 🤖.

1. Click on the "Connect" or "+ Add new authentication" button to add your OpenAI API key. You can get a FREE one key here .

2. Click on the "Connect" or "+ Add new authentication" button to give the function access to the GitHub repo to deploy the 🤖. That is to give access to the owner/repo in the environment variables. You'll be redirected to a new page where you must grant flows.network permission to the repo.

After that, click on the "Check" button to go to the flow details page. As soon as the flow's status became `running`, the PR summary GitHub bot is ready to give code reviews! The bot is summoned by every new PR or magic words (i.e., trigger_phrase) in PR comments.

## Next

Customize your own rules to review PR [in the code](https://github.com/flows-network/github-pr-summary) for your GitHub bot.


