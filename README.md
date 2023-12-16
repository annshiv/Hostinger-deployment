# Hostinger-deployment
A GitHub Action for hostinger deployment

## 1. Create Git repository for code
First thing you'll need to do is create a repo for your code on Github. Whether you choose to create a public or private repo, you'll have to do some different steps in the future, so be mindful.

![image](https://github.com/annshiv/Hostinger-deployment/assets/49332020/045eb530-62ff-440a-82e4-3d33499446a4)

Once you have the repo created, add, commit, and push your changes to the repo. There's many resources online to help you with this. But if this is your first time, the following commands should be all you need to do:

```
echo "# test-repo" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:sample/react.git #change to your Github repository
git push -u origin main
```

Once that's finished and you see your code in your repository, you're good to go to the next step!

## 2. Set up Git configuration on Hostinger
Next we'll set up the Git configurations for Hostinger. Back on Hostinger, you'll need to go to the GIT configuration settings for your website. So go to Hosting -> (your website) -> scroll to the Advanced section and select GIT.

<img width="271" alt="image" src="https://github.com/annshiv/Hostinger-deployment/assets/49332020/70a22a0c-0dc3-4520-b78d-2047efc52b88">

First, we will go to the Create a Repository section and add your git repo (private repo's should look like git@github.com:sample/react.git and public repo's should look like https://github.com/sample/react.git). For branch, put build. The reason for this is when we push our code to Github, we want the `build` of the code to be sent to Hostinger, not the code itself (the last step will cover this).

<img width="1102" alt="image" src="https://github.com/annshiv/Hostinger-deployment/assets/49332020/83e6b133-d820-4fcf-aa75-9ae2987f0e92">

If this branch does not exist, that's okay. If you currently have a branch for `build`, make sure you are ok with it being used to have the builds pushed to this branch for the future. Note: if you already have contents in your public_html folder, you will have to delete the contents inside, but we will quickly get everything back in there soon enough!
In the Manage Repositories section, click Auto Deployment and note the Webhook URL, since we will use it in the next step.

<img width="1106" alt="image" src="https://github.com/annshiv/Hostinger-deployment/assets/49332020/901d4dbb-aa39-4ebb-bc1f-65bb176186a8">

If your repo is private, continue with this next step, but if not you can skip to step 3.
Scroll up to the Private Git Repository and click the generate SSH Key button. Keep note of key, since we will use it in the next step.

## 3. Configure Github to push to Hostinger
Next, we'll set up your repository to send your builds to Hostinger whenever you push to your repo. First on Github, go to your settings on your repository and then select Webhooks. Paste the Webhook URL from the previous step into the payload URL input and leave the content type as application/x-www-form-urlencoded. You don't need to add a secret, and you can leave everything else as is (just make sure that the `Just the push event option` is selected. Select `Add webhook` and then you're almost done.

<img width="719" alt="image" src="https://github.com/annshiv/Hostinger-deployment/assets/49332020/c13b9e93-d1a5-4190-93c2-6692d4bc1bbf">

Now if your repo is private, you'll have to follow this next step, otherwise you can skip to the final part.
Now to successfully allow Hostinger to pull your build, you need to go to `Deploy keys` in the repo settings and paste the ssh key that was generated in the previous step. You can title it `Hostinger key` and it does not need write access, so you can leave the `allow write access` box unchecked.

<img width="750" alt="image" src="https://github.com/annshiv/Hostinger-deployment/assets/49332020/6b0f0275-2c6f-4eb3-8b9d-976bf89ab925">

Once you click `Add key` you're good to go to the final step.

## 4. Add Github action to repository
Lastly, we will set up Github Actions to build your code whenever you push to your repo. Now all you need to do is create a folder called `.github/workflows` from the root, and add a `publish.yml` file in there.

<img width="1437" alt="image" src="https://github.com/annshiv/Hostinger-deployment/assets/49332020/c6063db0-c3d5-45c0-83a9-365b235db8d6">

Post the following code into this file, and if you are not using the build branch, then make sure to change lines 26-27 to the name of the branch you plan to use:

```
name: Generate a build and push to another branch

on:
  push:
    branches:
      - main # The branch name your are commit the new changes

jobs:
  build:
    runs-on: ubuntu-latest
    name: Build and Push
    steps:
      - name: git-checkout
        uses: actions/checkout@v3

      - name: Install all dependencies
        run: npm install

      - name: Build
        run: npm run build # The build command of your project

      - name: Push
        uses: s0/git-publish-subdir-action@develop
        env:
          REPO: self
          BRANCH: build # The branch name where you want to push the assets
          FOLDER: build # The directory where your assets are generated
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub will automatically add this - you don't need to bother getting a token
          MESSAGE: "Build: ({sha}) {msg}" # The commit message
```

Once you add this, add, commit, and push the change.
Your pipeline should be fully set up so whenever you push a change to your main branch, this should automatically update your website on Hostinger!
