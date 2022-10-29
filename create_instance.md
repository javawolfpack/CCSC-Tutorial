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

When you first access the console you'll need to accept the terms and conditions, and should get a dialog like the following that you should accept to move forward:

![GCP First Run](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/firstrun.png "Figure 2: GCP First Run")

We'll now be prompted to select a project and we'll want to create a new project:

![GCP First Select Project](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/selectaproject.png "Figure 3: GCP First Select Project")

In the create a new project dialog you should give your project a name, like could say *ccsc-tutorial* for the project name. Make sure you select the *"Billing Account for Education"* in the billing account dropdown, that is the credit from the coupon you redeemed in a previous step. There is a chance it is called something different with the coupons in this tutorial. But once your configuration looks similar to below click **create** to create your project.

![GCP First Project](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/newproject.png "Figure 4: GCP First Project")

## Create first VM

So now we should be able to see the dashboard for our new project that looks something like the following:

![GCP Dashboard](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/dashboard.png "Figure 5: GCP Dashboard")

On the dashboard we can click directly on the **"Create a VM"** button to create a new VM. There is an alternative way to find this in the hamburger menu under compute instances as well. We will need enable the *Compute Engine API* to create VM instances on GCP. We should be directed to a page to do this when we clicked on the **"Create a VM"** button earlier that looks like this.

![GCP Enable API](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/enableAPI.png "Figure 6: GCP Enable API")

Once you've clicked on the **"Enable API"** button you'll a bit before being directed to the *Create an instance* dialog. Should look something like this:

![GCP Create Instance 1](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance1.png "Figure 7: GCP Create Instance 1")

You can now give this first instance a name, I would recommend something like *ccsc1* we will eventually have 3 of these instances. I have selected an *N1* series machine and the *n1-standard-1* machine type for the demo as these machines are the least expensive non-shared core instances. I've found this is extremely helpful when intially setting up GCP instances especially when in a classroom setting. After the nodes are setup and created you could change them to be an *E2* series machine of the *e2-micro* machine type and it would cost less than a single one of these *n1-standard-1* machines to run three of them continously. All the other defaults for the *Machine configuration* section can stick with the defaults. We'll move on to the boot disk settings now.

![GCP Create Instance 2](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance2_bootdisk.png "Figure 8: GCP Create Instance 2")

For the boot disk settings we will just change the image type to be *Ubuntu 22.04 LTS* for an *x86/64* system as GCP now supports arm instances, which our *N1* series machien is not. Next we'll need to expand the *Advanced options*:

### Advanced Options

![GCP Create Instance 3](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance3_advancedoptions.png "Figure 9: GCP Create Instance 3")

Once you click on the *Advanced options* dialog it will expand:

![GCP Create Instance 4](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance4_advanceddetail.png "Figure 10: GCP Create Instance 4")

In there there are two further dialogs we will want to configure regarding the networking and SSH key access. Let's work through the *Networking* details first:

![GCP Create Instance 5](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance5_advancednetwork.png "Figure 11: GCP Create Instance 5")

In the *Networking* dialog what we want to do is click into the *Networking interfaces* section. This is where we can setup static IPs so that the machines will be consistently at the same network locations even if powered off.

![GCP Create Instance 6](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance6_networkinterfaceedit.png "Figure 12: GCP Create Instance 6")

In the *Edit network interface* dialog the first thing we want to do is change the *Network Service Tier* to *Standard* for this example we do not need the *Premium* Tier networking and that is most often also what I've recommended to my students. Then we'll want to change the *Primary internal IP* settings:

![GCP Create Instance 7](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance7_networkreservestaticinternal.png "Figure 13: GCP Create Instance 7")

When you click on the *Primary internal IP* settings it creates a dropdown like the one above that we'll want to click to *Reserve static internal ip address*.

![GCP Create Instance 8](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance8_networkinternalstaticreservation.png "Figure 14: GCP Create Instance 8")

In the *Reserve a static internal IP address"* form you'll need to give your IP reservation a name. I decided to name it *ccsc1internal* as this is the internal static IP for our *ccsc1* VM instance. Make sure that the *Purpose* is set to *Non-shared* or the instance will not create as the *Shared* internal IPs are more for load balancing kind of operations related to web technologies. Once you click *Reserve* a fixed internal IP will be created, make sure to take note of this internal IP as we'll need it later.

### Optional Networking

It's an optional step but now that we're back in networking details, click on *External IPv4 address* and like with the internal IP click *Create IP Address* to launch the reservation form for an external IP address:

![GCP Create Instance 9](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance9_networkrservestaticexternal.png "Figure 15: GCP Create Instance 9")

Now you will get the *Reserve a new static IP address* dialog. You like before need to name your static IP reservation, which for this machine I named *ccsc1external*

![GCP Create Instance 10](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance10_networkexternalstaticreservation.png "Figure 16: GCP Create Instance 10")

### Optional SSH Key Access

Google Cloud Platform is amazing in that we can create these VM instances and interact with them entirely from a browser based terminal interface, which makes them an option for students on a Chromebook that need a Linux VM; however, this web terminal is not quite as responsive as a native terminal and you've SSH'd into the remote GCP instance. To get SSH access to these GCP instances you need to add public keys to the instance that are allowed access.

If we go back to the *Security* section under the *Advanced Options* we can see it has a further drop down to *Manage Access*.

![GCP Create Instance 11](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance11_securitydetail.png "Figure 17: GCP Create Instance 11")

After we click into the *Manage Access* drop down menu we can see a section to *Add manually generated SSH keys* this is where we can manually add SSH keys as items that we can use to access our VM instance. You could also setup project-wide SSH keys and then only need to check a box to reuse those same keys on future VMs.

![GCP Create Instance 12](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance12_securityaddkeyitem.png "Figure 18: GCP Create Instance 12")

When we click *Add Item* we'll be given a form box to paste in our public key contents.

![GCP Create Instance 13](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance13_addsshkey.png "Figure 19: GCP Create Instance 13")

At the end of your public SSH keys there is always a section that is *<username>@<hostname>* that represents who you are on the machine that created the key. GCP will generate a new user account with the same *username* as this part of your key, you can change this in the form and your keys will still work.

###

Now click *Create* and your first VM will be created.

![GCP Create Instance 14](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance14_create.png "Figure 20: GCP Create Instance 14")


## Repeat

Do this again and make two more GCP compute instances. When you created your first VM you would be sent to the GCP instances page, where you can just click the *Create Instance* button to make another instance

![GCP Create Instance 15](https://github.com/javawolfpack/CCSC-Tutorial/raw/main/assets/createinstance15_instances.png "Figure 21: GCP Create Instance 15")
