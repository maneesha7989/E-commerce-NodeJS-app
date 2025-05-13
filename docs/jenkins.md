# **Jenkins CI/CD Pipeline**

## **Access Jenkins**

1. Start the Jenkins Server
```bash
sudo systemctl start jenkins
```

2. Get your Instance IP Address

3. Access your Jenkins on the Browser
   - `http://IP-ADDRESS:8080/`

4. Get the Initial Password for Jenkins
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Copy the Password

5. Paste the Password on the Browser

6. Click on Install Suggested Plugins

7. Set your Username & Password for Jenkins

8. Continue Steps and you are on the Dashboard

> ![Jenkins UI](/docs/assets/jenkins-ui.png)

## **Plugins**

Navigate to **Manage Jenkins** > **Manage Plugins** > **Available** and install:

* Pipeline Stage View
* SonarQube Scanner

Restart Jenkins if needed after installation.

> [!NOTE]
> 
> These plugins are essential for visualizing pipeline stages and running code quality scans.

## **Integreation**

### SonarQube Setup

1. Make sure Docker is Running
```bash
sudo systemctl status docker
```

1. Run SonarQube Docker Image
```bash
docker run -d -p 9000:9000 --name sonarqube sonarqube:lts-community
```

1. Get SonarQube on Browser
   - `http://IP-ADDRESS:9000`
   - **Default Username:** admin
   - **Default Password:** admin

2. You will be on SonarQube UI
3. Go to the **Administration** > **Security** > **User** you will see a **Tokens** Branch along with this **:-**, click on it.
4. Type somename you want like `jenkins` and it will give you a token
5. Save the token or Paste it somewhere.
6. Create a SonarQube Webhook by getting on the **Administration** > **Configuration** > **Webhook** and Create a new Webhook.
7. Enter a name again like `jenkins-webhook` and the other section you will add
   - `http://IP-ADDRESS:8080/sonarqube-webhook/`
8.  Your SonarQube is now Ready!

## **Credentials**

Go to **Jenkins Dashboard** > **Manage Jenkins** > **Credentials** > **global** and **Add Credentials**

### SonarQube Credentials

1. Change the Type of Credentials from Username & Password to Secret Text from the Dropdown Box
2. Enter the Token which you copied from SonarQube
3. Enter the name of the Token as `sonarQubeToken`. (description if needed)
4. Click Crete and you have now added SonarQube Token.

### Docker Hub Credentials

1. Again do the following Step by getting to the Credentials Dashboard and Add new Credentials
2. Add your DockerHub Username on the first part.
3. Enter your DockerHub Token on the Second Part
> [!CAUTION]
>
> You can get your DockerHub Token from DockerHub by getting to your **DockerHub Setting** click on **Personal Access Token (PAT)**, create a new one by naming it and you will get your Credentials. Make sure you give it permission like **Read and Write**.

4. Name it something like `dockerHubCredentials` 
5. You have now created the DockerHub Credentials

### Email Credentials

1. Go to your Google Account Settings at [myaccount.google.com](https://myaccount.google.com/)
2. Navigate to **Security** > **2-Step Verification** > **App passwords**
   - Note: You must have 2-Step Verification enabled for your Google account
3. Select **Mail** as the app and **Other** as the device
4. Enter a name (e.g., "Jenkins") and click **Create**
5. Google will generate a 16-character app password - copy this password
6. In Jenkins, go to **Manage Jenkins** > **Credentials** > **global** > **Add Credentials**
7. Select **Username with password** from the dropdown
8. Enter your full Gmail address as the username
9. Enter the app password you copied as the password
10. Set ID as `emailCredentials` and add a description
11. Click **Create**

## **Setup Necessary Packages**

### Email Configuration

1. Go to **Manage Jenkins** > **System**
2. Scroll down to **Email Notification** section
3. Enter `smtp.gmail.com` as SMTP server
4. Check **Use SMTP Authentication**
5. Enter your Gmail address as username
6. Select the email credentials you created from the dropdown
7. Check **Use SSL**
8. Set SMTP Port to `465`
9. Enter your Gmail address as the default sender email
10. Click **Test configuration** to verify everything works
11. Click **Save** at the bottom of the page

### SonarQube Integration with Jenkins

1. **Configure SonarQube Scanner Tool:**
   - Go to **Manage Jenkins** > **Global Tool Configuration**
   - Find the **SonarQube Scanner** section and click **Add SonarQube Scanner**
   - Name it `SonarQubeScanner` (this name will be used in your pipeline)
   - Select **Install automatically** and choose the latest version
   - Click **Save**

2. **Configure SonarQube Server:**
   - Go to **Manage Jenkins** > **System**
   - Find the **SonarQube servers** section
   - Click **Add SonarQube**
   - Name: `SonarQubeScanner` (this name will be used in your pipeline)
   - Server URL: `http://IP-ADDRESS:9000` (replace with your actual SonarQube URL)
   - Server authentication token: Select the `sonarQubeToken` credential you created earlier
   - Click **Save**

Now your Jenkins instance is properly configured to use SonarQube for code quality analysis in your pipelines.

## **Shared Library**

> [!NOTE]
>
> Only add this if you want to use my `Jenkinsfile`, if you have your Jenkinsfile then don't do this or check my Shared Library and Configure it according to your Jenkins Pipeline.

To run the pipeline, you'll need to configure the shared library:

- Go to **Manage Jenkins** > **System**
- Scroll to **Global Pipeline Libraries** section and click **Add**
- Library Name: `jenkinsLibrary` (use this exact name in your Jenkinsfile)
- Default version: `main`
- Retrieval method: Select **Modern SCM**
- Select **Git** and enter repository URL:
  ```
  https://github.com/Abdullah-0-3/sharedLibJenkins.git
  ```
- Click **Save**

> [!TIP]
> The shared library contains pre-configured pipeline steps that make your Jenkinsfile cleaner and more maintainable.

## **Running Jenkins Pipeline**

To set up your pipeline from repository:

1. From Jenkins dashboard, click **New Item**
2. Enter a name for your pipeline (e.g., `tws-ecommerce-pipeline`)
3. Select **Pipeline** and click **OK**
4. In the configuration page:
   - Under **Pipeline**, select **Pipeline script from SCM**
   - Select **Git** as SCM
   - Repository URL: `https://github.com/Abdullah-0-3/tws-e-commerce-app.git`
   - Specify branch: `*/devops`
   - Script Path: `Jenkinsfile` (must exist in your repository)
5. Click **Save**
6. Click **Build Now** to run the pipeline

> [!TIP]
> The pipeline will use the Jenkinsfile from the devops branch of your repository.

Your Jenkins is now fully functional and you can enjoy the Magic of Jenkins.
