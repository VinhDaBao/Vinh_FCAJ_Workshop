---
title : "5.4.6. Test the CI/CD Automation"
weight : 6
---

Now that our CodePipelines are deployed and fully linked to our GitHub repository, it's time to see the magic of GitOps in action! 

In this section, we will make a small code change on our local machine, push it to GitHub, and watch AWS automatically build and deploy it to our live web application without any manual intervention.

## Step 1: Make a Local Code Change

Open your PubliCast monorepo in your local IDE (e.g., VSCode). Let's simulate a small UI change request on the Frontend.
Specifically, open the file `frontend/src/pages/auth/Login.jsx` and find the `LeftPanel` logo text. Let's change the word **StreamHub** to **StreamHubbb**:

```jsx
// frontend/src/pages/auth/Login.jsx
<div className="flex items-center gap-2 mb-auto">
  <div style={{ width: 32, height: 32, background: "#FFF", borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center" }}>
    <Wifi size={16} color="#0A0A0A" />
  </div>
  {/* Change StreamHub to StreamHubbb */}
  <span style={{ color: "#FFF", fontSize: 16, fontWeight: 500 }}>StreamHubb</span>
</div>
```

## Step 2: Commit and Push to GitHub

Open your terminal and use Git to commit this small change, then push it to the branch that CodePipeline is watching.

```bash
git add .
git commit -m "feat: change logo text to StreamHubbb for pipeline testing"
git push origin develop
```

## Step 3: Watch the Pipeline (In Progress)

Because of the CodeStar webhook, AWS instantly detects your `git push`.
1. Go to the **AWS Console** -> **CodePipeline**.
2. Click on your Frontend Pipeline.
3. You will see the **Source** stage pull the new code, and then the **Build** stage will light up in blue (Status: **In Progress** - building the React app).

{{< img "images/Workshop/services/pipeline-progress.png" "AWS Console - Pipeline In Progress" >}}

## Step 4: Pipeline Success

Wait a few minutes for CodeBuild to finish compiling the UI (`npm run build`) and pushing it to S3/CloudFront. Once completed, all stages in the CodePipeline will turn green (**Succeeded**).

{{< img "images/Workshop/services/pipeline-success.png" "AWS Console - Pipeline Success" >}}

## Step 5: Verify the Live Changes

The ultimate test! Open a new tab in your browser and navigate to the Login page on your Frontend live domain.

You should immediately see the text **StreamHub** has been changed to **StreamHubbb**! Everything happens fully automated on the live internet. You have successfully deployed a production-grade CI/CD pipeline.

{{< img "images/Workshop/services/login-streamhubbb.png" "Live Web App - StreamHubbb UI" >}}
