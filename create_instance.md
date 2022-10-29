# Creating Instances

## Redeem GCP Coupon

This will be handed out at the start of the tutorial. You'll need to go to the following link to redeem the coupon:

[https://console.cloud.google.com/education](https://console.cloud.google.com/education)

I do not recommend redeeming this on your campus gmail if that is what you have as I know my campus google services are configured to not allow google cloud usage by default.

### Redeeming Coupon

Clicking on that link will povide a dialog that looks like below:

![GCP Redemption](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/GCP_application.png "Figure 1: GCP Redemption")

In this form you should make sure you have your name and enter the GCP coupon that was handed out at the start of this tutorial.

## Create Project

Now you can go to the Google Cloud Platform Console:

[https://console.cloud.google.com/](https://console.cloud.google.com/)

When you first access the console you'll need to accept the terms and conditions, and should get a dialogue like the following that you should accept to move forward:

![GCP First Run](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/firstrun.png "Figure 2: GCP First Run")

We'll now be prompted to select a project and we'll want to create a new project:

![GCP First Select Project](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/selectaproject.png "Figure 3: GCP First Select Project")

In the create a new project dialogue you should give your project a name, like could say *ccsc-tutorial* for the project name. Make sure you select the *"Billing Account for Education"* in the billing account dropdown, that is the credit from the coupon you redeemed in a previous step. There is a chance it is called something different with the coupons in this tutorial. But once your configuration looks similar to below click **create** to create your project.

![GCP First Project](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/newproject.png "Figure 4: GCP First Project")

## Create first VM

So now we should be able to see the dashboard for our new project that looks something like the following:

![GCP Dashboard](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/dashboard.png "Figure 5: GCP Dashboard")

On the dashboard we can click directly on the **"Create a VM"** button to create a new VM. There is an alternative way to find this in the hamburger menu under compute instances as well. We will need enable the *Compute Engine API* to create VM instances on GCP. We should be directed to a page to do this when we clicked on the **"Create a VM"** button earlier that looks like this.

![GCP Enable API](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/enableAPI.png "Figure 6: GCP Enable API")



