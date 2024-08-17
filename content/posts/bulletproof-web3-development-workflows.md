--- 
draft: false
date: 2024-08-01T20:47:19+02:00
title: "Bulletproof Web3 Development Workflows"
description: "Secure Web3 development workflows by setting up YubiKey for GPG commit signing."
slug: "bulletproof-web3-development-workflows"
authors: ["Oussama Amri"]
tags: ["Web3 Security", "Blockchain Development", "DevSecOps"]
categories: []
externalLink: ""
series: []
---

Learn how to secure your Web3 and blockchain development workflows by setting up YubiKey for GPG commit signing, adding your GPG key to GitHub, and implementing automated verification with GitHub Actions. This step-by-step guide ensures the integrity of your codebase by preventing unauthorized and malicious commits.

## TL;DR

In the rapidly evolving landscape of Web3 and blockchain development, security is paramount. As software companies working in these domains strive to build decentralized, immutable systems, the integrity of the codebase becomes a critical aspect of their operations. With Git being the backbone of most software version control, ensuring that the code committed to repositories is secure and tamper-proof is more important than ever.

### The Role of Git in Web3

Git is the lifeblood of modern software development, enabling collaboration, version control, and efficient project management. For Web3 and blockchain companies, Git repositories house the very code that powers decentralized applications (dApps), smart contracts, and other critical blockchain infrastructure. Any compromise in the integrity of this code could have devastating consequences, including the potential loss of millions in assets, breaches of trust, and irreparable damage to reputations.

Given the high stakes, it’s not enough to rely on traditional security measures like passwords or even two-factor authentication (2FA). These can be bypassed by sophisticated attackers, potentially leading to unauthorized code changes. This is where hardware-based security measures, like the use of YubiKey for GPG signing, come into play.

## Hardware-Based Security with YubiKey

YubiKey, a hardware authentication device, adds an additional layer of security by requiring physical touch to sign commits with GPG (GNU Privacy Guard). This process ensures that only authorized developers can sign off on code changes, making it significantly more difficult for attackers to push malicious code even if they gain access to a developer's Git account.

The benefits of using YubiKey for GPG signing in Web3 and blockchain development are manifold:

1. **Preventing Unauthorized Access:** Even if an attacker gains access to a developer’s GitHub account through phishing or other means, they cannot push code to the repository without physically possessing the YubiKey.
   
2. **Ensuring Code Integrity:** Every commit is cryptographically signed with a GPG key that is protected by the YubiKey, ensuring that the commit can be traced back to an authorized developer.

3. **Enhancing Developer Accountability:** GPG signing provides a clear audit trail of who signed what, making it easier to trace back any issues to their source.

4. **Mitigating Supply Chain Attacks:** By requiring hardware authentication for code commits, you reduce the risk of supply chain attacks where malicious code could be introduced into your project.


### Part 1: Setting Up YubiKey for GPG Commit Signing

Before we dive into generating keys and configuring Git, it's crucial to set up your YubiKey securely. This includes changing the default User and Admin PINs, generating the GPG key directly on the YubiKey, and requiring a physical touch for each commit signing to enhance security.

#### Step 1: Install Required Software

Ensure you have the following software installed on your machine:

- **GnuPG**: For generating and managing GPG keys.
- **YubiKey Manager (ykman)**: For configuring your YubiKey.
- **YubiKey GPG Tools**: Allows you to use your YubiKey for GPG signing.

On Ubuntu or Debian-based systems, install these tools using:
```bash
sudo apt-get install gnupg2 yubikey-manager scdaemon
```

On macOS, use Homebrew:
```bash
brew install gnupg gpg yubikey-manager ykman ykpers pinentry pinentry-mac 
```

#### Step 2: Change the User and Admin PINs

It's essential to change the default User and Admin PINs on your YubiKey to prevent unauthorized access.

1. **Insert your YubiKey** into your computer.

2. **Set a Reset Code** (optional but recommended) to allow PIN reset:
   ```bash
   ykman openpgp reset
   ```

3. **Open a terminal** and run the following command to change the User PIN (default: 123456):
   ```bash
   ykman openpgp access change-pin
   ```

4. **Change the Admin PIN** (default: 12345678) using the following command:
   ```bash
   ykman openpgp access change-admin-pin
   ```

#### Step 3: Generate the GPG Key on the YubiKey

Next, generate the GPG key directly on your YubiKey. This ensures that the private key never leaves the hardware, providing maximum security.

1. **Access the YubiKey GPG interface**:
   ```bash
   gpg --card-edit
   ```

2. **Enter the following commands** within the GPG prompt to generate the key:
   - Type `admin` and press **Enter**.
   - Type `generate` and press **Enter**.
   - For our purposes, there is no need to make a backup, please answer ‘n’ here.
   > You will be prompted to enter your USER PIN

3. **Set the key** to **0** in order to not expire.

4. **Enter your name, email address, and a strong passphrase**.
   - You will be prompted to enter your USER PIN

The GPG key is now generated on your YubiKey.

#### Step 4: Require Physical Touch for GPG Signing

To enforce a higher level of security, configure your YubiKey to require a physical touch every time it is used for GPG signing.

- **Run the following command** to configure touch requirements:
   ```bash
   ykman openpgp keys set-touch SIG ON
   ```
  > You will be prompted to enter your ADMIN PIN

This command configures your YubiKey to require a physical touch for every GPG signature operation, ensuring that no commit can be signed without your explicit consent.

#### Step 5: Configure Git to Use Your YubiKey GPG Key

Configure Git to use your newly generated GPG key for signing commits:

1. **List your GPG keys** to find the key ID:
   ```bash
   gpg --list-keys
   ```

2. **Configure Git** to use this key for signing commits:
   ```bash
   git config --global user.signingkey <your-key-id>
   git config --global commit.gpgSign true
   ```

#### Step 6: Test Your Setup

Create a test commit to ensure that your GPG key is being used for signing:

- **Make a test commit**:
   ```bash
   git commit -S -m "Test commit with YubiKey GPG signing"
   ```

During this process, you will be prompted to **enter your USER PIN** and **touch your YubiKey**, confirming that the setup is working correctly.

#### Step 7: Export Your Public Key

Finally, you need to export your GPG public key so it can be shared with others and added to your GitHub profile:

- **Export your GPG public key**:
   ```bash
   gpg --armor --export <your-key-id> > <your-name>.asc
   ```

This key can now be added to your GitHub profile and shared with your SecOps team to ensure your commits are verified. This setup ensures that your commits are cryptographically signed, secure, and verified using your YubiKey, providing an additional layer of security crucial for Web3 and blockchain development.

### Part 2: Adding the GPG Key to Your GitHub Profile

**Step 1: Copy Your GPG Public Key**

Open the exported `<your-name>.asc` file and copy its contents.

```bash
cat <your-name>.asc
```

**Step 2: Add the GPG Key to GitHub**

1. **Log in to GitHub.**
2. **Navigate to Settings -> SSH and GPG keys.**
3. **Click "New GPG key".**
4. **Paste your GPG public key into the text box.**
5. **Click "Add GPG key".**

Your commits will now show as verified on GitHub.

### Part 3: Pushing the Public Key to a SecOps Repository

**Step 1: Clone the SecOps Repository**

Clone your organization's SecOps repository where public keys are stored:

```bash
git clone https://github.com/your-org/secops.git
```

**Step 2: Add Your Public Key**

Navigate to the repository folder and add your public key file:

```bash
cp ~/path-to/<your-name>.asc secops/keys/<your-username>.asc
```

**Step 3: Commit and Push the Changes**

Commit and push your public key to the repository:

```bash
git add keys/<your-username>.asc
git commit -m "Add GPG public key for <your-username>"
git push origin main
```

---



## Automated Verification with GitHub Actions

To further bolster security, it’s essential to implement automated workflows that verify the integrity of commits. For instance, a GitHub Actions workflow can be set up to automatically verify that all commits are signed with a YubiKey-generated GPG key. If a commit is unsigned or the signature is untrusted, the workflow can flag the commit and notify the appropriate teams, preventing unverified code from being merged into the main codebase.

This kind of automated verification acts as a final gatekeeper, ensuring that even if an attacker manages to bypass other security measures, their efforts will be thwarted by the lack of a valid hardware-generated signature.


### Part 1: Automated Verification with GitHub Actions

Now, let's walk through the GitHub Actions YAML file for automated verification of commits signed with GPG.

#### YAML File Breakdown

```yaml
name: Check GPG Signatures

on:
  pull_request:
    types: [opened, synchronize]
```
- **Purpose**: This action triggers when a pull request (PR) is opened or updated.
- **Use Case**: Ensures that all commits in the PR are signed with a valid GPG.

```yaml
jobs:
  check-signatures:
    runs-on: ubuntu-latest
```
- **Job**: `check-signatures` runs on the latest version of Ubuntu.
- **Environment**: This is a Linux environment, which is commonly used for CI/CD pipelines.

```yaml
    steps:
    - name: Checkout main repository
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
```
- **Step 1**: Checks out the code from the repository so that the action can inspect the commits.

```yaml
    - name: Checkout internal secops repository
      uses: actions/checkout@v3
      with:
        repository: your_org/secops
        path: secops
        token: ${{ secrets.GITHUB_TOKEN }}
```
- **Step 2**: Checks out an internal repository where public keys are stored (like the SecOps repository mentioned earlier).

```yaml
    - name: Load all public GPG keys
      run: |
        echo "Loading GPG keys from secops repository..."
        for key in secops/keys/*.asc; do
          echo "Importing key: $key"
          gpg --import "$key"
        done
        echo "All GPG keys loaded."
```
- **Step 3**: Imports all the GPG public keys from the SecOps repository, ensuring that the signatures can be verified against the public keys stored.


```yaml
    - name: Check GPG signatures
      id: gpgcheck
      run: |
        BASE_BRANCH=${{ github.event.pull_request.base.ref }}
        echo "Base branch: $BASE_BRANCH"
        COMMITS=$(git log origin/$BASE_BRANCH..HEAD --pretty=format:'%h|%G?|%GK|%an|%ae|%s' --skip=1)
        UNMATCHED_COMMITS=""
        UNMATCHED_COUNT=0
        while IFS= read -r line; do
          COMMIT_ID=$(echo $line | cut -d '|' -f 1)
          SIGNED=$(echo $line | cut -d '|' -f 2)
          GPG_KEY_ID=$(echo $line | cut -d '|' -f 3)
          AUTHOR=$(echo $line | cut -d '|' -f 4)
          EMAIL=$(echo $line | cut -d '|' -f 5)
          MESSAGE=$(echo $line | cut -d '|' -f 6-)
          echo "Commit ID: $COMMIT_ID, Author: $AUTHOR, Email: $EMAIL, Signed: $SIGNED, Key ID: $GPG_KEY_ID"
          if ([ -n "$SIGNED" ] && [ "$SIGNED" != "G" ] && [ "$SIGNED" != "U" ]); then
            UNMATCHED_COMMITS="$UNMATCHED_COMMITS| $COMMIT_ID | $GPG_KEY_ID | &#x1F7E5; No | $AUTHOR | $MESSAGE |\n"
            UNMATCHED_COUNT=$((UNMATCHED_COUNT + 1))
          fi
        done <<< "$COMMITS" 
        COMMENT_BODY="## YubiKey GPG Signature Check\n\n"
        if [ $UNMATCHED_COUNT -gt 0 ]; then
          echo "Unmatched commits found: $UNMATCHED_COUNT"
          COMMENT_BODY="$COMMENT_BODY### ❌ Signature Mismatch\n\nSome commits have a signature mismatch with your YubiKey GPG key. Please ensure you sign your commits using your YubiKey.\n\n| Commit ID | Key ID | Matched | Author | Commit Message |\n| --------- | --------- | ------ | ------ | -------------- |\n$UNMATCHED_COMMITS\n"
          echo -e "$COMMENT_BODY" > check_results.md
        else
          echo "All commits are matched."
        fi 
        echo "unmatched_count=$UNMATCHED_COUNT" >> $GITHUB_OUTPUT
```
- **Step 5**: This script checks each commit in the PR for its signature. It ensures that the commits are either signed with a GPG key and that the signature is trusted. If any commits are unsigned, unverified, or untrusted, they are flagged.

### Part 2: Committing Results to the PR

```yaml
    - name: Comment on PR
      id: prcomment
      if: steps.gpgcheck.outputs.unmatched_count > 0
      uses: actions/github-script@v7
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          const fs = require('fs');
          const commentHeader = '## YubiKey GPG Signature Check';
          const commentBody = fs.readFileSync('check_results.md', 'utf-8');

          // Get existing comments on the PR
          const { data: comments } = await github.rest.issues.listComments({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
          });

          console.log("Existing comments fetched.");

          // Find the existing comment by the bot
          const botComment = comments.find(comment => comment.user.login === 'github-actions[bot]' && comment.body.startsWith(commentHeader));

          if (botComment) {
            // Update the existing comment
            console.log("Updating existing comment...");
            await github.rest.issues.updateComment({
              comment_id: botComment.id,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: commentBody,
            });
            console.log('Comment updated');
            core.setOutput('action_result', 'updated');
          } else {
            // Create a new comment
            console.log("Creating a new comment...");
            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: commentBody,
            });
            console.log('Comment created');
            core.setOutput('action_result', 'created');
          }  
```
- **Step 6**: This step posts a comment on the PR with the details of any unsigned, unverified, or untrusted commits.

### Part 3: Sending Alerts to Slack

```yaml
    - name: Send Slack Notification
      if: steps.gpgcheck.outputs.unmatched_count > 0 && steps.prcomment.outputs.action_result == 'created'
      run: |
        PR_LINK="${{ github.event.pull_request.html_url }}"
        REPO_NAME="${{ github.repository }}"
        PR_TITLE="${{ github.event.pull_request.title }}"
        UNMATCHED_COUNT="${{ steps.gpgcheck.outputs.unmatched_count }}"
        AUTHOR="${{ github.event.pull_request.user.login }}"

        echo "Sending Slack notification..."
        echo "PR Link: $PR_LINK"
        echo "Repository: $REPO_NAME"
        echo "PR Title: $PR_TITLE"
        echo "Number of Unmatched Commits: $UNMATCHED_COUNT"
        echo "Author: $AUTHOR"

        curl -X POST -H 'Content-type: application/json' --data "{
          \"attachments\": [
            {
              \"color\": \"warning\",
              \"pretext\": \":lock_with_ink_pen: *YubiKey GPG Signature Check*\",
              \"fields\": [
                {
                  \"title\": \"Scan Result\",
                  \"value\": \"One or more commits are unmatched in ${REPO_NAME}\",
                  \"short\": false
                },
                {
                  \"title\": \"Number of Unmatched Commits\",
                  \"value\": \" :x: ${UNMATCHED_COUNT}\",
                  \"short\": true
                },
                {
                  \"title\": \"PR Name\",
                  \"value\": \"${PR_TITLE}\",
                  \"short\": false
                },
                {
                  \"title\": \"Author\",
                  \"value\": \"${AUTHOR}\",
                  \"short\": false
                }
              ],
              \"actions\": [
                {
                  \"type\": \"button\",
                  \"text\": \"View Findings\",
                  \"url\": \"${PR_LINK}\"
                }
              ]
            }
          ]
        }" ${{ secrets.SLACK_GH_SECURITY_WEBHOOK }}
```
- **Step 7**: If the action creates a new comment on the PR, a Slack notification is sent to alert the team that there are unsigned, unverified, or untrusted commits.


## Conclusion

By following this guide, you have set up a robust security workflow that ensures only signed, verified, and trusted commits are merged into your codebase. This is especially critical for Web3 and blockchain companies, where the integrity of the code is paramount. The use of hardware-based security with YubiKey and automated verification with GitHub Actions adds multiple layers of protection, significantly reducing the risk of unauthorized or malicious code changes.

For Web3 and blockchain companies, the stakes of security breaches are incredibly high. By incorporating hardware-based security with tools like YubiKey for GPG signing, and implementing automated verification workflows, companies can significantly enhance the integrity of their codebases. This approach not only protects against unauthorized code changes but also ensures that every commit can be trusted, thereby safeguarding the decentralized, transparent principles that Web3 and blockchain technology are built upon.