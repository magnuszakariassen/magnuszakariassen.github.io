# Github Actions workshop step-by-step guide

## Prerequisites

1. From your personal github account create a new public repository with the name www.your_github_username.github.io. 

2. You should now see three options on how to set up your repository. Click on the option `Import code` and use the repository url 
https://github.com/acntech/workshop-github-actions.git. Click `Begin import`.

Great, now we're ready to start!

## Step 1: The basics 

You can find the workflow file in `.github/workflows/build-pipeline.yml`. Github Actions will automatically detect all workflows that are located in this folder.

Let us first start with a little introduction to how a workflow file is set up. The file is written in `YAML` format and stands for *YAML ain’t markup language* (a recursive acronym), which emphasizes that YAML is for data, not documents. Under you can see an example of a `.yaml`/`.yml` file

```yaml
name: This is a workflow

on:
  push:
    branches:
    - master
  pull_request:
    branches:
    - master

jobs:
  print-comment:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "This is a comment"
```

This workflow has the name `This is a workflow` and has only one job that has the name `print-comment`. For more complex workflows you would usally have multiple different `jobs`, and these run in parallell by default Both the name of the workflow and the name of the jobs can be whatever you want. The `on` parameter tells us that this workflow will be triggered every time we push changes or create a pull request to the `master` branch.

One of the nice things about GitHub Actions is that it doesn't just support running builds on Linux hosts, or in containers, but it also provides virtual machines running on Windows and macOS. So if you're building cross-platform applications, you can easily verify your code in different OSs. To specify the host type, you indicate that with the `runs-on` parameter for a job. Here, we are running on a Linux VM by using `
ubuntu-latest`.

Next in the workflow is `steps` which are the building blocks of a job. These are processes that are run in the environement you specified above, and has access to both the filesystem and workspace. The final term we are going to introduce here is `run`. Run triggers command-line programs using the operating system's shell, such as the unix-command `echo`, or trigger a python command such as `pip install`.

## Step 2: Let's build and deploy our code 🚀

Below you can see an example of a job that first checkouts our code, sets up `Node.js` (with stated version), installs all necessary dependencies, then builds our code before deploying it to GitHub pages. GitHub pages lets you easily turn GitHub repositories into websites, which is exactly what we are going to do here.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 14.x
        uses: actions/setup-node@v2
        with:
          node-version: '14.x'
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@4.1.4
        with:
          branch: gh-pages
          folder: dist
```
The parameter `uses` selects an action to run as part of a step in your job. An action is just a reusable unit of code which can be defined in your own repository, in a public repository, or in a published Docker container image. 

In the example above, the deploy step will create a new branch named `gh-pages` that will contain our built code used in deploying our website.

**Task: In the file `.github/workflows/build-pipeline.yml`, fill in the necessary information in order to deploy your code every time you push to the `main` branch. Push your changes to `main` to see that the workflow runs green in your `Actions` page. Once green, your website will be available at www.your_github_username.github.io.**
## Step 3: Oh no! Something is wrong with our code 🐛

We now want to add the step of automatically testing our code before we build it. When setting up multiple jobs in a workflow, the jobs run independently of each other, in parallel. Usually, that's ideal. Your jobs will run as soon as machines are available to execute them.

But sometimes you want to be able to set up jobs that depend on other jobs. For example, you might have some services that you want to test against. But to save money, you only want to run those services when you're actually running tests. So you might want to have a job that starts your services, a job that runs your tests, and then a job that stops your services.

To specify dependencies between jobs, you can use the `needs` parameter to indicate what jobs rely on the completion of other jobs. In the example below, `job1` must complete successfully before `job2` begins, and `job3` waits for both `job1` and `job2` to complete.

```yaml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

**Task: Add a job that first checkouts the code, sets up `Node.js`, installs dependencies, and automatically runs all tests using the command `npm test` before deploying. Make sure that we don't deploy if any test fails. Verify that both your jobs are running successfully by checking the workflow your `Actions` page.**

**Discussion point: What are the benefits of adding this step to our workflow?**

Ever heard about linting before? Well, linting is what makes your code readable, less dependent on who wrote the code, and generally prettier to look at. If you want to learn more about why you should lint, check out [this](https://www.perforce.com/blog/qac/what-lint-code-and-why-linting-important) blogpost.

```yaml
      - name: Run typescript lint and format check
        run: npm run lint:ts
      - name: Run styles lint and format check
        run: npm run lint:css
```
  
**Task: Add the steps above for running typescript and css linting to your existing job that contains the testing step. Verify that everything is running successfully by checking the workflow in your `Actions` page.**

## Step 4: Adding secrets

You'll often need things like tokens or passwords in deployment scenarios. For instance, if your application needs to log into you bank account to retrieve some information you don't want to have your social security number explicitly stated in your code. This is sensitive information which we normally would want to hide using `secrets`.

To set up a secret, go to your `Repository Settings` page, then select `Secrets`. Your secret's name will be used in your workflow to reference the data, and you can place the secret itself in the value.
  
To use that secret, you can reference it using the secrets context within your workflow. If you had a secret named `PASSWORD`, you could reference that as `${{secret.PASSWORD}}`.

**Task: Create a job `secret` that runs on a Linux VM and saves the secret you've created to the environment variable `PASSWORD`.**
  
## Step 5: Ok, so now we have added some must have steps to our workflow. Let's explore! 🍀

You're now able to automatically check linting and testing before deploying your code. And you didn't have to do anything except push your code!

Now, let's see what fun we can do! And there are sooo many options. For instance,

## Step 6: Sending an email notification every time a job fails and/or succeeds. 📫
  
### Set a timing for when a deploy should be set. ⏰

## Step-<INPUT> You are becoming a pro, time to explore Github Marketplace
In the same way there are libraries for almost any usecase when you write code, there are thousand of Github Actions already created for you to utilize. To continue to improve your repository go to the [Github Marketplace](https://github.com/marketplace?category=&query=sort%3Apopularity-desc&type=actions&verification=), find an action you like, and try to incorporate it into you repository.